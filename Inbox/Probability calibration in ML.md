# Probability calibration in ML

Capture: calibrating the output probabilities an ML model gives.

- When a model says 0.9, is it right 90% of the time? Calibration methods: Platt scaling, isotonic regression, temperature scaling.
- Interesting link: relates to softmax outputs — see [[Numerical Instability in Softmax Cross-Entropy]] for what happens to probabilities when logits grow unboundedly.

## Questions / Things to explore

- Why do modern nets produce overconfident probabilities? How does temperature scaling fix it?
