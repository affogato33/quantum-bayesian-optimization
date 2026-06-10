# Quantum-Assisted Bayesian Optimization for Hyperparameter Tuning

A study of variational quantum circuits as acquisition function optimizers
in Bayesian hyperparameter optimization. The project benchmarks a quantum
Bayesian optimization (QBO) method against classical GP-BO, Optuna TPE,
random search, and a language-model surrogate, on hyperparameter tuning
tasks for SVM and Random Forest classifiers.

## Method

The quantum optimizer combines a classical Gaussian process surrogate with
a variational quantum circuit (VQC) trained via the parameter-shift rule
to concentrate its measurement distribution on configurations with high
expected improvement. The VQC uses a hardware-efficient ansatz with CRZ
gates placed at parameter boundaries to capture cross-parameter
correlations.

- 6 qubits, 2 layers, 46 trainable circuit parameters
- Parameter-shift gradients with 512 shots, averaged over 2 repeats
- UCB acquisition during the first 5 observations, expected improvement
  thereafter, computed over a Matern-5/2 Gaussian process
- Batch sampling from the trained quantum distribution

## Key Findings

**VQC concentration is measurable and reproducible.** Across five random
seeds on a 64-configuration SVM search space, the VQC distribution
collapsed from the maximum 6.00 bits of Shannon entropy to a mean of
0.86 bits, a concentration on roughly two configurations out of 64.

**Barren plateau scaling matches theory.** Gradient variance decays
exponentially with qubit count, dropping from 2.96e-03 at 4 qubits to
1.41e-04 at 10 qubits across 30 random parameter samples, consistent
with Cerezo et al. (2021).

**Noise resilience under hardware-realistic error rates.** The method
finds the global optimum under depolarizing error probabilities from
0.000 to 0.020, covering the regime of current superconducting hardware.

**Quantum BO matches classical methods at sufficient budget.** At 50
evaluations on the SVM benchmark, quantum BO, GP-BO, Optuna TPE, and
random search all find the optimum (validation accuracy 0.7204).

**Classical methods outperform quantum BO at tight budgets.** At 10
evaluations, Optuna TPE averages 0.6993 while quantum BO averages 0.3785,
reflecting the overhead of training the VQC before its concentration
benefit can amortize.

## Benchmarks

**SVM on digits with low-data, noisy-label regime.** 50 training samples,
15% label noise. Search space: 8 values of C in [1e-3, 1e4] crossed with
8 values of gamma in [1e-4, 1e3], 64 configurations total. The score
distribution across the full grid has mean 0.22 and standard deviation
0.22, with only 5 out of 64 configurations scoring above 0.7 — a
genuinely sparse optimum.

**Random Forest on breast cancer.** A flatter landscape where almost all
configurations score within 5% of optimum, included as a control.

## Tooling

- **Qiskit and Qiskit-Aer:** circuit construction, parameter-shift
  gradients, shot-based evaluation, depolarizing noise simulation,
  barren plateau diagnosis
- **scikit-learn:** SVM and Random Forest objectives, Gaussian process
  surrogate with Matern-5/2 kernel
- **scikit-optimize:** classical GP-BO baseline with expected improvement
- **Optuna:** TPE optimizer baseline
- **HuggingFace Transformers:** distilgpt2 surrogate conditioned on prior
  configuration-score pairs as text prompts, with direct control of
  decoding temperature
- **PyTorch, NumPy, SciPy, Matplotlib:** numerics and visualisation

## References

1. Cerezo, M., Arrasmith, A., Babbush, R., Benjamin, S. C., Endo, S.,
   Fujii, K., McClean, J. R., Mitarai, K., Yuan, X., Cincio, L. and
   Coles, P. J. (2021). Variational quantum algorithms.
   *Nature Reviews Physics* 3, 625-644.
2. Schuld, M., Bergholm, V., Gogolin, C., Izaac, J. and Killoran, N. (2019).
   Evaluating analytic gradients on quantum hardware.
   *Physical Review A* 99, 032331.
3. Shahriari, B., Swersky, K., Wang, Z., Adams, R. P. and de Freitas, N. (2016).
   Taking the human out of the loop: A review of Bayesian optimization.
   *Proceedings of the IEEE* 104(1), 148-175.
4. Snoek, J., Larochelle, H. and Adams, R. P. (2012). Practical Bayesian
   optimization of machine learning algorithms. *NeurIPS*.
