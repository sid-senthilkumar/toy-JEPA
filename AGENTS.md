# AGENTS.md — Spark Agent Operating Manual

## Goal

Train a LeWM latent world model on alanine dipeptide MD trajectories so that the learned latent space recovers the C7eq and C7ax metastable basins — without phi/psi supervision — and passes all validation criteria in `docs/OBJECTIVE.md`.

## Start here

1. Read `docs/CONSTRAINTS.md` — this is the contract. Some things are fixed; most things are yours.
2. Read `docs/OBJECTIVE.md` — this is the definition of done.
3. Read `docs/BACKGROUND.md` if you want orientation on the method.
4. Implement everything inside `src/`. You own the layout entirely.

## Hard constraints (summary — see `docs/CONSTRAINTS.md` for the full contract)

- **Two-term LeWM objective only:** `L = L_pred + lambda * SIGReg(Z)`. No other terms.
- **Anti-collapse is SIGReg only.** No EMA target network, no stop-gradient, no frozen or pretrained encoder. These are the exact heuristics LeWM was designed to eliminate. If you find yourself reaching for them, stop — you have left the LeWM recipe.
- **SE(3)-invariant input featurization.** Internal coordinates (dihedrals and/or interatomic distances). Raw unaligned Cartesian coordinates are disallowed.
- **Collapse must be monitored and reported throughout training**, not just at the end.
- **Toy scope only.** No protein/ATLAS work, no ESM encoder backbone.

## Your freedoms

You own all of the following — make the calls, don't ask for defaults:

- Encoder and predictor architecture and sizes
- Latent dimensionality
- `lambda`, number of SIGReg random projections, optimizer, learning rate, batch size, schedule
- Prediction horizon (how many steps ahead)
- Exact internal-coordinate featurization
- Train/validation split, number of epochs, early stopping criterion
- Tooling beyond PyTorch and mdshare
- Code organization inside `src/`

## Environment facts (known, not for you to solve)

- **Hardware:** NVIDIA DGX Spark — GB10 Grace–Blackwell, **aarch64/ARM64**, unified memory, CUDA.
- **ARM64 gotcha:** some Python wheels are not on PyPI for aarch64; use the NVIDIA package index or build from source where needed.
- **Scale:** data fits in memory; single GPU; a passing run should complete in minutes, not hours.

## Definition of done

See `docs/OBJECTIVE.md`. All five criteria must hold. Save required artifacts to `results/`.

## Reproducibility

Fix a random seed. Log your config (hyperparameters, featurization choices, seed) alongside results. The reviewer will check that a re-run from the logged config produces consistent outputs.
