# toy-JEPA

This repo develops a **JEPA-style latent world model** — specifically the LeWorldModel (LeWM) architecture (Maes et al., 2026) — on a molecular dynamics toy system. An encoder maps MD frames to a latent; a predictor rolls the latent forward in time; the model is trained with a two-term objective (latent prediction loss + SIGReg anti-collapse regularizer). The toy system is **alanine dipeptide**: a 22-atom molecule whose slow dynamics are fully captured by two dihedral angles (phi/psi). Success is a learned latent that recovers the known C7eq/C7ax metastable basins without ever seeing phi/psi as input.

This is the **toy/sanity-check stage only** of a larger LeWM-in-bio-ML effort. Protein-scale trajectories, ATLAS data, and ESM encoder backbones are explicitly out of scope here.

## Status

Skeleton awaiting implementation. `src/` is empty.

## Where to look

- **Hardware agent:** start with `AGENTS.md`.
- **Human reviewer:** start with `docs/SUPERVISOR_NOTES.md`.
- **Background and method:** `docs/BACKGROUND.md`.
- **Success criteria:** `docs/OBJECTIVE.md`.
- **What is fixed vs. free:** `docs/CONSTRAINTS.md`.
