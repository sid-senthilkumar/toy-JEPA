# LeWM Alanine Dipeptide Toy

This repo tests a **JEPA-style latent world model** — specifically the LeWorldModel (LeWM) architecture (Maes et al., 2026) — on a molecular dynamics toy system. An encoder maps MD frames to a latent; a predictor rolls the latent forward in time; the model is trained with a two-term objective (latent prediction loss + SIGReg anti-collapse regularizer). The toy system is **alanine dipeptide**: a 22-atom molecule whose slow dynamics are fully captured by two dihedral angles (phi/psi). Success is a learned latent that recovers the known C7eq/C7ax metastable basins without ever seeing phi/psi as input.

This is the **toy/sanity-check stage only** of a larger LeWM-in-bio-ML effort. Subsequent phases (protein-scale, ATLAS trajectories, ESM encoder backbone) are explicitly out of scope here.

## Four-layer model

| Layer | Who | Responsibility |
|-------|-----|----------------|
| Chat | Human supervisor | Constraints encoded in this repo's docs |
| Docs | Claude Code (this scaffold) | Goals, constraints, definition of done |
| Code | Spark agent | All architecture, hyperparameter, and implementation decisions |
| Review | Human engineer | Verify agent output against the docs |

## Status

Skeleton awaiting the Spark agent. `src/` is empty. The agent reads `AGENTS.md` first.

## Where to look

- **Spark agent:** start with `AGENTS.md`.
- **Human reviewer:** start with `docs/SUPERVISOR_NOTES.md`.
- **Background and method:** `docs/BACKGROUND.md`.
- **Success criteria:** `docs/OBJECTIVE.md`.
- **What is fixed vs. free:** `docs/CONSTRAINTS.md`.
