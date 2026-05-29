# SUPERVISOR_NOTES.md — Reviewer Checklist

This document is for the **human engineer** reviewing the Spark agent's finished work. Run through this checklist against the agent's code and results before signing off.

---

## Checklist

### Method integrity

- [ ] **SIGReg only.** Search the code for: `ema`, `momentum`, `target_encoder`, `stop_grad`, `detach` (in the target-branch role), `frozen`, `pretrained`. None of these should appear as an anti-collapse mechanism. A `detach` for an unrelated reason (e.g., logging) is fine — confirm context. Silent reversion to the old JEPA recipe is the single likeliest deviation.

- [ ] **Two-term objective.** The training loss is `L_pred + lambda * SIGReg(Z)`. No reconstruction term, no contrastive term, no extra regularizers that weren't in the spec.

- [ ] **SE(3)-invariant inputs.** Confirm the model receives internal coordinates (dihedrals, distances, or equivalent). Grep for raw Cartesian coordinate arrays being passed directly to the encoder without alignment.

### Results quality

- [ ] **Collapse diagnostic is a trace, not a point.** The results include a logged curve of the collapse metric over training epochs. A single end-of-run value is insufficient.

- [ ] **Latent recovers C7eq/C7ax without supervision.** Open the latent-vs-Ramachandran figure. The two basins should be visually separable. Confirm by reading the featurization code: phi and psi must not appear as model inputs or labels.

- [ ] **Baseline comparison is quantitative.** A numeric metric (clustering overlap, mutual information, or equivalent) against the known metastable assignment, not just a visual claim.

- [ ] **Rollout diagnostic is present.** Multi-step rollouts were evaluated. The diagnostic shows the rollouts don't immediately diverge.

### Reproducibility and scale

- [ ] **Config is logged.** A config file in `results/` records hyperparameters, featurization choices, and random seed.

- [ ] **Runtime is toy-appropriate.** Check the logged wall-clock time or timestamps. A run that took many hours at this scale warrants investigation.

- [ ] **Seed is fixed.** The config includes a seed, and re-running from that config produces consistent outputs.

---

## Red flags — stop and investigate if you see these

| Red flag | What it likely means |
|----------|----------------------|
| Near-zero loss paired with a collapsed latent (low variance, rank-1 or rank-2 latent) | SIGReg is not functioning or `lambda` is zero / too small |
| Results that look unusually clean (perfect basin separation, zero divergence in rollouts) | Check that phi/psi were not inadvertently supplied as features |
| EMA encoder, momentum encoder, stop-gradient, or frozen backbone in the code | Agent reverted to pre-LeWM JEPA recipe — non-compliant |
| Protein/ATLAS trajectories, ESM backbone, or multi-residue systems | Scope violation — out of bounds for the toy stage |
| Multi-term objective (VICReg-style or reconstruction loss) | Objective is not the LeWM two-term form |
| Wall-clock runtime trending toward hours | Likely a data, batching, or device placement bug at toy scale |

---

## References for reviewer context

- LeWM paper: Maes et al. 2026, arXiv:2603.19312 — describes SIGReg and explains why EMA/stop-grad are unnecessary.
- `docs/BACKGROUND.md` — in-repo primer on the method and the toy system.
- `docs/OBJECTIVE.md` — the full five-criterion definition of done.
- `docs/CONSTRAINTS.md` — the fixed-vs-free contract the agent was given.
