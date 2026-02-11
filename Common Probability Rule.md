# Common Probability Rules & Identities

A compact cheat sheet of the most commonly used probability rules, with the **product rule** as the anchor.

---

## 1) Product Rule (Chain Rule)

### Two variables
$$p(x,y) = p(x \mid y)\,p(y) = p(y \mid x)\,p(x)$$

### Multiple variables (chain rule)
$$p(x_1,\dots,x_n) = \prod_{i=1}^n p(x_i \mid x_1,\dots,x_{i-1})$$

---

## 2) Sum Rule / Marginalization (Law of Total Probability)

### Discrete \(y\)
$$p(x) = \sum_y p(x,y)$$
$$p(x) = \sum_y p(x \mid y)\,p(y)$$

### Continuous \(y\)
$$p(x) = \int p(x,y)\,dy = \int p(x \mid y)\,p(y)\,dy$$

---

## 3) Bayes’ Rule

$$p(y \mid x) = \frac{p(x \mid y)\,p(y)}{p(x)}$$

General form:
$$p(A \mid B) = \frac{p(B \mid A)\,p(A)}{p(B)}$$

---

## 4) Conditional Probability Definition

$$p(A \mid B) = \frac{p(A \cap B)}{p(B)}, \quad p(B)>0$$

Equivalently:
$$p(A \cap B) = p(A \mid B)\,p(B)$$

*(This is the origin of the product rule.)*

---

## 5) Independence Rules

### Independence
$$X \perp Y \iff p(x,y)=p(x)p(y)$$
$$\iff p(x \mid y)=p(x),\quad p(y \mid x)=p(y)$$

### Conditional independence
$$X \perp Y \mid Z \iff p(x,y \mid z)=p(x \mid z)p(y \mid z)$$

---

## 6) Law of Total Expectation (Tower Rule)

$$\mathbb{E}[X] = \mathbb{E}\big[\mathbb{E}[X \mid Y]\big]$$

**Intuition:** average within groups, then average across groups.

---

## 7) Law of Total Variance

$$\mathrm{Var}(X) = \mathbb{E}[\mathrm{Var}(X \mid Y)] + \mathrm{Var}(\mathbb{E}[X \mid Y])$$

**Interpretation:**  
total variance = within-group variance + between-group variance.

---

## 8) Inclusion–Exclusion (Union Rule)

Two events:
$$p(A \cup B) = p(A)+p(B)-p(A \cap B)$$

If \(A,B\) are disjoint:
$$p(A \cup B) = p(A)+p(B)$$

---

## 9) Complement Rule

$$p(A^c) = 1 - p(A)$$

Useful:
$$p(\text{at least one}) = 1 - p(\text{none})$$

---

## 10) Union Bound (Boole’s Inequality)

$$p\!\left(\bigcup_i A_i\right) \le \sum_i p(A_i)$$

Common in theoretical ML bounds.

---

## 11) Markov & Chebyshev Inequalities

### Markov (for nonnegative \(X\))
$$P(X \ge a) \le \frac{\mathbb{E}[X]}{a}$$

### Chebyshev
$$P(|X-\mathbb{E}[X]| \ge k\sigma) \le \frac{1}{k^2}$$

---

# Mental Map (How They Connect)

- **Conditional definition → Product rule**  
- **Product rule + Marginalization → Bayes rule**  
- **Conditionals + grouping → Total expectation/variance**

---

✅ This set covers most needs in:
- Machine Learning  
- Statistics  
- Probabilistic modeling  
- Bayesian inference
