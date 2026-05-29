# OBJECTIVE.md — Definition of Done

A run passes when **all five** of the following criteria hold. Partial passes are not passes.

---

## 1. No collapse

The collapse diagnostic (latent variance, effective rank, or SIGReg term magnitude — agent's choice of diagnostic) shows that the latent distribution remained high-variance and full-rank throughout training. This must be a logged trace over training, not a single end-of-run value. A run where loss is low but the latent has collapsed is a **failure**, not a success.

**Required artifact:** collapse diagnostic curve saved to `results/`.

---

## 2. Latent recovers metastable structure

Project the learned latent to 2D (e.g., PCA or UMAP) and overlay with the known phi/psi Ramachandran landscape. The two metastable basins — **C7eq** and **C7ax** — must be visually distinguishable in latent space. phi and psi must not have been provided as model input or used in any supervision signal.

**Required artifact:** latent-vs-Ramachandran figure saved to `results/`.

---

## 3. Baseline sanity check

Compare the latent-space clustering against a reference assignment of metastable states (e.g., the known Ramachandran basin boundaries, a simple MSM, or a k-means clustering in phi/psi space). The comparison should confirm that the latent structure is physically meaningful, not an artifact of the architecture or training noise.

**Required artifact:** quantitative comparison metric (e.g., clustering overlap, mutual information, or equivalent) in `results/`.

---

## 4. Rollout stays on-manifold

Run multi-step latent rollouts from held-out starting frames. The rollouts must not diverge into nonsense — the predictor should produce latents that remain within the learned distribution and track plausible dynamics. Report a rollout stability metric (e.g., latent norm over rollout steps, or latent log-likelihood under the trained model).

**Required artifact:** rollout diagnostic saved to `results/`.

---

## 5. Reproducible and toy-scale

- A fixed random seed and the full config (hyperparameters, featurization, seed) are logged.
- A re-run from the logged config produces consistent outputs.
- The run completed in toy-appropriate wall-clock time (order of minutes). A run trending toward many hours signals a scope or implementation problem.

**Required artifact:** config file (JSON, YAML, or equivalent) saved to `results/`.

---

## Results directory layout (minimum)

```
results/
├── config.*                  logged hyperparameters and seed
├── collapse_diagnostic.*     collapse metric trace over training
├── latent_ramachandran.*     latent-vs-phi/psi figure
├── baseline_comparison.*     quantitative metastable-state comparison
└── rollout_diagnostic.*      multi-step rollout stability
```

Additional figures, logs, or checkpoints are welcome alongside these.
