# AGENTS.md — Operating Manual

## Goal

Train a LeWM latent world model on alanine dipeptide MD trajectories so that the learned latent space recovers the C7eq and C7ax metastable basins — without phi/psi supervision — and passes all validation criteria in `docs/OBJECTIVE.md`.

## Reading order

1. `docs/CONSTRAINTS.md` — the contract. Read this first.
2. `docs/OBJECTIVE.md` — the definition of done.
3. `docs/BACKGROUND.md` — method primer. Optional; read if you want orientation.

Then implement everything inside `src/`. You own the layout entirely.

## Hard constraints (summary — full contract in `docs/CONSTRAINTS.md`)

- **Two-term LeWM objective only:** `L = L_pred + lambda * SIGReg(Z)`. No other terms.
- **Anti-collapse is SIGReg only.** No EMA target network, no stop-gradient, no frozen or pretrained encoder. These are the exact heuristics LeWM was designed to eliminate. If you find yourself reaching for them, stop — you have left the LeWM recipe. This is the likeliest silent deviation, because the older JEPA recipes dominate the training distribution.
- **SE(3)-invariant input featurization.** Internal coordinates (dihedrals and/or interatomic distances). Raw unaligned Cartesian coordinates are disallowed.
- **Collapse must be monitored and reported throughout training**, not just at the end.
- **Toy scope only.** No protein/ATLAS work, no ESM encoder backbone.

## Your freedoms

These are yours — make the calls, state no defaults:

- Encoder and predictor architecture and sizes
- Latent dimensionality
- `lambda`, number of SIGReg random projections, optimizer, learning rate, batch size, schedule
- Prediction horizon
- Exact internal-coordinate featurization
- Train/validation split, epochs, early stopping
- Tooling beyond PyTorch and `mdshare`
- Code organization inside `src/`

## Environment facts

- **Hardware:** NVIDIA DGX Spark — GB10 Grace–Blackwell, **aarch64/ARM64**, unified memory, CUDA.
- **ARM64 gotcha:** some Python wheels are not on PyPI for aarch64; the NVIDIA package index or a source build may be needed.
- **Scale:** data fits in memory; single GPU; a passing run completes in minutes, not hours. A run trending toward many hours signals a problem.

## Definition of done

See `docs/OBJECTIVE.md`. All five criteria must hold.

## Reporting

Report state by writing to the repo:

- `results/` — artifacts: the latent-vs-Ramachandran figure, collapse trace, metrics, logged config.
- `results/STATUS.md` — keep this current: what ran, pass/fail against each criterion in `docs/OBJECTIVE.md`, and any blocker. Plain text, updated as the run progresses.
