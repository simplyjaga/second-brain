---
title: Numerical Instability in Softmax + Cross-Entropy
tags: [machine-learning, deep-learning, numerical-stability, softmax, cross-entropy, paper-notes]
source: arXiv 2610.31642
authors: Jonathan T. Barron, Dan Gnanapragasam, Michael Tschannen (Google DeepMind)
date-read: 2026-11-15
status: learning
---

# Numerical Instability in Softmax + Cross-Entropy

## What is it?

Standard softmax/cross-entropy code has two float-precision bugs. When the model is very confident, the computer rounds away part of the gradient. The paper fixes this by reordering the math, not by changing the function.

## Background: softmax and its gradient

Softmax turns raw logits $z_i$ into probabilities: $p_i = \frac{e^{z_i}}{\sum_j e^{z_j}}$.

- Cross-entropy loss (one-hot label): $L = -\log(p_{correct})$; library form: $L = -z_c + \log\sum_j e^{z_j}$ (log-sum-exp).
- Backprop gradient at the logits: $g_i = p_i - y_i$.
- Gradients **sum to zero**: $\sum g = \sum p - \sum y = 1 - 1 = 0$, because softmax is shift-invariant. This fact powers the fix.

Worked example z=[20,0,0]:

| Step | Result |
|---|---|
| $e^{20}$, $e^0$, $e^0$ | 485,165,195 / 1 / 1 |
| Denominator | 485,165,197 |
| $p_0$ | 0.99999999588 |
| $p_1, p_2$ | ≈ 2.06e-9 |

```python
z = z - z.max()           # subtract max for overflow safety
p = np.exp(z) / np.exp(z).sum()
```

## The two bugs (float32)

- **Softmax Collapse:** correct logit much bigger → $p_{correct}$ rounds to exactly 1.0 → $g_{correct} = p - 1 = 0$. Wrong classes keep small gradients, so gradients no longer sum to zero.
- **Softmax Cancellation (distillation):** student and teacher probabilities computed separately then subtracted; near-equal values lose precision, so the student gets nonzero gradient even when it matches the teacher exactly.

Why float32 breaks $p_0$: floats keep ~7 *significant digits*, not decimal places. Small numbers ($2.06\times10^{-9}$) are accurate — the exponent holds the position. Near 1, the digits are all 9s and the useful part (gap from 1) sits past the cutoff:

```
0.9999999 | 9588
 ↑ kept     ↑ thrown away  (this tail holds the information "4.1e-9 below 1")
```

| | True value | Stored in float32 |
|---|---|---|
| p0 | 0.99999999588 | 1.0 |
| p1 | 2.06e-9 | 2.06e-9 |
| p2 | 2.06e-9 | 2.06e-9 |

| Node | Computed | True |
|---|---|---|
| g0 = p0 − 1 | **0** | −4.12e-9 |
| g1 | 2.06e-9 | 2.06e-9 |
| g2 | 2.06e-9 | 2.06e-9 |
| Sum | 4.12e-9 ✗ | 0 |

Loss too: true ≈ 4.1e-9, but $-\log(1.0) = 0$.

## The fix (same numbers)

- **Gradient:** skip the broken subtraction $p_0 - 1$; use sum-to-zero: $g_0 = -(g_1 + g_2) = -4.12e-9$. Small numbers store accurately → exact. Node 0 still gets its gradient.
- **Loss:** margin form with `log1p`: $L = \log(1 + \sum_{j \ne c} e^{z_j - z_c})$ → $\log(1 + 2e^{-20}) \approx 4.1e-9$. The naive form computes $-20 + 20.0000000041$, rounding to 0.
- **Distillation:** compute student−teacher difference stably so equal logits give exactly zero gradient.
- Same answer as usual in normal cases, correct in extreme cases — safe everywhere.

```python
g = p.copy()
g[c] = -(p.sum() - p[c])   # instead of p[c] - 1
```

## When does the bug appear?

Only when $p_{correct}$ is so close to 1 that float32 can't tell it apart — logit margin ≳ 17–18. z=[5,0,0] → $g_0$=−0.01325: fine. Creeps in late in training as logits keep growing:

| p0 | True g0 | Rounding error as % of g0 |
|---|---|---|
| 0.99 | −1e-2 | ~0.00001% |
| 0.99999 | −1e-5 | ~0.6% |
| 0.9999999 | −1e-7 | ~50% |
| 0.99999999588 | −4e-9 | 100% |

## What the paper reports (toy benchmarks)

- Eliminates the "Slingshot" loss-spike in grokking experiments.
- Speeds up grokking with weight decay (mod 31: 94±5% → 100±0%).
- Prevents accuracy collapse without weight decay (mod 97: 61%→23% vs fixed 85%).
- Stops student drifting from teacher in distillation.
- Logit growth becomes bounded by equilibrium between weight decay and Adam's ε.

> [!warning] Caveat
> Results are from small toy tasks; large-scale impact unverified.

## Things that confused me (Q&A)

> [!question] Q1. Why does a number close to 1 lose precision but a small number doesn't?
> Floats keep ~7 significant digits from the first nonzero digit. Small numbers spend those digits on real information; near 1 the digits are all 9s and the gap-from-1 info sits past the cutoff. The real problem is subtracting $p_0 - 1$ on an already-rounded value.

> [!question] Q2. If we skip p0, how does node 0 get a gradient?
> We skip the broken calculation $p_0 - 1$, not node 0. It gets $g_0 = -(g_1 + g_2)$, which equals $p_0 - 1$ at infinite precision. All nodes get gradients.

> [!question] Q3. Why should gradients sum to zero? I learned probabilities sum to one. (Zero, not one.)
> $\sum g = \sum p - \sum y = 1 - 1 = 0$ — probabilities summing to 1 is exactly why gradients sum to 0. Adding a constant to every logit leaves softmax and loss unchanged, so gradients must cancel. Push up on the correct class = total push down on others.

## Takeaways

- The bug is numerical (float rounding), not a flaw in the math — specific to softmax+CE, not all losses.
- Trigger: $p_{correct}$ ≈ 1 (logit margin ≳ 17–18 in float32), or student ≈ teacher in distillation.
- Fix: correct-class gradient from −sum of the others; loss via margin form with `log1p`.
- Mental model: floats are good at small numbers, bad at tiny differences from 1.

## Related concepts

## Projects / Journeys

## Sources

- arXiv 2610.31642 — Barron, Gnanapragasam, Tschannen (Google DeepMind)

## Questions / Things to revisit

- Reproduce both bugs myself in PyTorch/JAX with float32 and z=[20,0,0]?
