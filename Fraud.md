# AMAZON L5 DATA SCIENTIST — MODELING CASE #2 (Fraud / Abuse)

Context
You’re on an Amazon Payments / Marketplace Risk team. A small fraction of orders are fraudulent
(stolen cards, account takeovers, triangulation, etc.). 

You need a model to score orders in real-time
and decide whether to (a) approve, (b) step-up verify, or (c) decline.

(This is a classic Amazon-style DS/ML case: imbalanced classification + cost-sensitive decisions + latency constraints.
Fraud detection is a common Amazon ML domain; AWS publishes multiple fraud-detection solution patterns.) :contentReference[oaicite:0]{index=0}



## 1) The Case Prompt (what the interviewer gives you)

Build a real-time fraud model that outputs a risk score at checkout within 50ms p95.

Data (12 months of historical orders):
- order_id, checkout_ts, payment_amount, currency
- account_id, account_age_days, number_of_cards_on_file
- device_id, ip_prefix, geo (coarse), login_velocity (count in last 1h/24h)
- shipping_address_hash, billing_address_hash, address_mismatch_flag
- item_category, item_price, quantity, gift_card_flag
- seller_id (if marketplace), fulfillment type
- label: fraud = 1 if chargeback / confirmed fraud within 60 days, else 0
- optional: whether an order was previously “challenged” (OTP / 3DS / manual review)

Constraints / realities:
- fraud rate ~0.2% (very imbalanced)
- decisions are cost-sensitive (false negatives expensive; false positives hurt CX & conversion)
- label delay: fraud label can arrive weeks later
- policy feedback loop: model decisions affect which orders become labels

Tasks:
A) Describe your modeling approach end-to-end.
B) Specify leakage controls and how you handle delayed labels.
C) Choose offline metrics and how you set thresholds for actions (approve / challenge / decline).
D) Explain how you would evaluate online impact.
E) List 5 failure modes + monitoring.

(These are common expectations in Amazon DS interview prep resources: imbalance handling, tradeoffs, debugging.) :contentReference[oaicite:1]{index=1}



## 2) “Strong L5” Solution Outline

### A) End-to-end approach
**1) Framing**
- Score: p(fraud | checkout-time features).
- Convert score → policy via 3-way decision:
  - approve if score < t1
  - step-up verify (OTP/3DS/manual review) if t1 ≤ score < t2
  - decline if score ≥ t2

**2) Split strategy (time-based + label delay)**
- Use time-based splits AND a label availability window:
  - Train on orders from months 1–8 with labels fully matured (e.g., at least 60 days old).
  - Validate on months 9–10 matured.
  - Test on months 11–12 matured.
- Why: avoids training on partially-labeled data (right-censoring) and reduces optimistic bias.

**3) Model candidates**
- Baseline: regularized logistic regression (fast, interpretable, strong baseline).
- Likely final: Gradient-boosted trees (LightGBM/XGBoost/CatBoost)
  - Handles non-linearities and sparse categorical encodings well; fast inference for tabular data.
- If you have rich graphs (account ↔ device ↔ address ↔ card): consider graph features
  (e.g., shared device counts, connected component risk), but keep inference budget in mind.
  AWS even showcases graph-based fraud solution patterns conceptually (Neptune ML). :contentReference[oaicite:2]{index=2}

### B) Leakage + delayed labels
**1) Checkout-time feature gating**
- Only features known at checkout_ts.
- Exclude anything post-order like delivery status, customer support contacts, refunds after purchase, etc.

**2) Label delay handling (matured labels)**
- Train only on orders old enough that fraud outcomes are “complete.”
- Optionally: treat as a survival / time-to-event problem later, but binary matured labels are fine for first pass.

**3) Feedback loop / selective labels**
- If some orders are challenged/declined, you may never observe true fraud outcomes.
- Strategy:
  - Keep a random “golden set” of traffic that follows current policy to preserve comparability.
  - Track policy-driven censoring explicitly (“challenged/declined” flags) and analyze bias.

### C) Metrics + thresholding (cost-sensitive + imbalanced)
Offline:
- PR-AUC (more informative than ROC-AUC under heavy imbalance)
- Recall at fixed precision (e.g., “recall when precision ≥ X”)
- Cost-weighted utility:
  - Expected cost = C_FN * FN + C_FP * FP + C_review * (# challenged)
  - Choose (t1, t2) to maximize expected profit / minimize expected loss given ops capacity
- Calibration (Brier / reliability) since score → policy is sensitive to miscalibration

### D) Online evaluation (A/B test)
Randomize at user/account level to reduce interference.
Treatment: new score + new thresholds or routing.
Control: existing rules/model.

Primary outcomes:
- Fraud loss rate (chargeback $ / GMV), fraud count
- Approval rate & conversion (guardrail; too many FPs hurts business)
- Customer friction metrics: challenge rate, OTP success, abandonment
- Manual review load and precision

### E) 5 failure modes + monitoring
1) Concept drift (fraud tactics shift)
- Monitor PR-AUC & calibration weekly; retrain cadence; drift detectors on key features.
2) Segment collapse (new region/device types)
- Track metrics by geo, device, seller cohort, payment amount buckets.
3) Data quality / pipeline issues
- Sudden distribution shifts; missingness spikes; schema changes.
4) Threshold mis-setting under peak events
- Dynamic thresholds with capacity constraints; fallback rules.
5) Adversarial behavior / gaming
- Rate-limit features that can be manipulated; monitor “unknown device” spikes; anomaly alerts.

(These themes match common Amazon DS interview expectations and “how interviewers assess case performance.”) :contentReference[oaicite:3]{index=3}


## 3) Modeling-heavy follow-ups (practice)

Q1) “What features matter most?”
- velocity features (login/order counts), device/address reuse counts, mismatch flags,
  account age, historical chargeback rate for account/device/address clusters.

Q2) “How do you handle imbalance?”
- class weights, downsample negatives, focal loss (if NN), and always evaluate with PR metrics + costs.

Q3) “How do you ensure explanations for ops?”
- SHAP on tree model + stable top drivers per decision bucket; keep features human-interpretable.

Q4) “What’s your deployment plan?”
- feature store with point-in-time correctness; online service; monitoring dashboards; rollback thresholds.
