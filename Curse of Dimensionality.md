# Curse of Dimensionality — Summary Notes

## 1) What “curse of dimensionality” means
As the number of features (dimension) $p$ increases, data become **sparse** in the input space. Many learning methods rely on the idea that “nearby points are similar,” but in high dimensions **nearby points are hard to find** unless the sample size $N$ is extremely large.

---

## 2) Why the curse happens (core reasons)

### A) Neighborhoods stop being “local”
For inputs uniformly distributed in a $p$-dimensional unit hypercube $[0,1]^p$, suppose we expand a hypercube neighborhood around a target point to capture a fraction $r$ of the data.

- Volume of a hypercube with edge length $e$ is $e^p$.
- To capture fraction $r$ of the volume (and thus data), we need:

$$e^p = r \quad \Rightarrow \quad e = r^{1/p}$$

In high dimensions, $r^{1/p}$ is large:
- $p=10$: $e_{10}(0.01)\approx 0.63$, $e_{10}(0.1)\approx 0.80$.

So to capture only 1%–10% of data, the neighborhood must cover 63%–80% of each feature’s full range.  
These neighborhoods are no longer “local,” so local averaging (like kNN) becomes biased.

If we reduce $r$ to keep the neighborhood small, we average fewer points, increasing variance.

**Result:** high dimension forces a trade-off:
- large neighborhood $\Rightarrow$ non-local $\Rightarrow$ higher bias  
- small neighborhood $\Rightarrow$ too few samples $\Rightarrow$ higher variance

---

### B) Most points lie near the boundary (edge effect)
| ![Image 1](https://github.com/user-attachments/assets/2fa296df-bf1d-4894-95a8-3cda23d223f3) | ![Image 2](https://github.com/user-attachments/assets/09cf2567-abd1-4229-948e-cfa5fc973c6c) |
| :---: | :---: |
| **Figure 1** | **Figure 2** |

For $N$ points uniformly distributed in a $p$-dimensional unit ball, even the nearest neighbor to the origin can be far.  
In high $p$, points concentrate in a thin shell near the boundary.

**Why this hurts:** prediction near edges is harder because you often **extrapolate** rather than **interpolate**:
- interior point: neighbors surround you (interpolation)
- boundary point: neighbors mostly on one side (extrapolation)

---

### C) Function complexity can explode with $p$
If the true target function $f(x_1,\dots,x_p)$ involves **high-order interactions** among many variables, the space of possible behaviors becomes extremely large.  
To estimate such a function with the same accuracy as in low dimensions, the needed training size $N$ often grows very rapidly (commonly described as “exponentially” with $p$) for flexible nonparametric learning.

---

## 3) Bias–variance perspective
- **Nonparametric methods** aim for low bias by being flexible, but in high $p$ they suffer high variance (few local neighbors) unless $N$ is huge.
- **Simple parametric models** are stable (lower variance) because they estimate a fixed number of parameters, but they may be biased if the true $f$ is complex.

---

## 4) Effect on parametric vs non-parametric models

### Non-parametric models (more sensitive to the curse)
Examples: kNN, kernel regression, KDE, Gaussian processes, RBF SVM, many tree-based methods (often treated as nonparametric).

**Why sensitive:** they rely on local neighborhoods / similarity in $p$-D space.
- In high dimension, neighborhoods must be large to include enough points $\Rightarrow$ lose locality (bias).
- Keeping neighborhoods small $\Rightarrow$ too few points (variance).

**Net:** more likely to break down when $p$ is large and $N$ is not enormous.

---

### Parametric models (less sensitive, but can be biased)
Examples: linear regression, logistic regression, LDA/QDA (given distributional form), fixed-architecture neural nets, fixed-degree polynomial regression.

**Why less sensitive:** model complexity is controlled by a fixed set of parameters; they do not require dense local coverage of the space.

**Main risk:** model mismatch (bias). If the true function contains strong nonlinearities/interactions, a too-simple parametric form cannot represent it well.

---

## 5) Key takeaway
- The curse of dimensionality is largely about **sparsity**: in high dimensions, “close neighbors” are not truly close unless $N$ is extremely large.
- This particularly harms **nonparametric** local methods (kNN-style) through a bias–variance trap.
- **Parametric** models are usually more stable in high $p$ but can suffer **bias** if the assumed functional form is too restrictive.
