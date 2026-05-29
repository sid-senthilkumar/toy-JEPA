# CONSTRAINTS.md — The Contract

This document states what is **fixed** and what is **free**. It is the contract the implementation must satisfy.

---

## Fixed — do not change these

These constraints define "doing LeWM on the toy." Changing any of them means you are no longer building what this project is testing.

### 1. Two-term LeWM objective

The training objective is exactly:

```
L = L_pred + lambda * SIGReg(Z)
```

`L_pred` is the latent prediction loss. `SIGReg(Z)` is the anti-collapse regularizer. No other terms. A VICReg-style multi-term expansion, an added reconstruction loss, or any other objective augmentation means you have left the LeWM recipe.

### 2. SIGReg is the only anti-collapse mechanism

This is the constraint most likely to be silently violated, so it is stated emphatically:

**Do not use an EMA target network. Do not use stop-gradient. Do not use a frozen or pretrained encoder.**

These are the exact heuristics LeWM was built to remove. The entire point of SIGReg is that they are not needed. If training feels unstable and you find yourself reaching for one of these, the correct response is to investigate your SIGReg implementation and `lambda` — not to add a target network.

### 3. SE(3)-invariant input featurization

Model inputs must be internal coordinates: dihedrals and/or interatomic distances (or equivalent SE(3)-invariant features). Raw Cartesian coordinates without alignment are **disallowed** — they make the prediction target unstable under rotation and translation of the molecule, which is unphysical noise.

### 4. Collapse must be monitored throughout training

A collapse diagnostic (latent variance, effective rank, SIGReg term magnitude, or equivalent — your choice) must be computed and logged across training epochs, not just at the end. A run with low loss but a collapsed latent is a failure. Reporting only final loss is insufficient.

### 5. Toy scope only

This repo covers the alanine dipeptide toy stage. Do not introduce:
- Protein-scale trajectories (ATLAS or equivalent)
- An ESM encoder backbone
- Any multi-residue system

Those belong to a later phase of the project. Their presence here is out of scope and will cause the reviewer to flag the run.

---

## Free — the implementation owns these

The following decisions are explicitly yours. Do not wait for guidance on them; make the calls and document your choices in the logged config.

- **Encoder architecture and size.** MLP, transformer, graph network, convolutional — whatever fits the featurization.
- **Predictor architecture and size.** Same latitude.
- **Latent dimensionality.**
- **`lambda`** (SIGReg weight), **number of random projections** in SIGReg, optimizer, learning rate, batch size, and learning rate schedule.
- **Prediction horizon.** How many steps ahead the predictor is trained to target.
- **Exact internal-coordinate featurization.** Which dihedrals, which distances, any normalization.
- **Train/validation split, number of epochs, early stopping criterion.**
- **Tooling.** PyTorch is assumed; everything else (MDAnalysis, mdtraj, PyEMMA, Lightning, Hydra, etc.) is your call.
- **Code organization inside `src/`.** The directory is empty; structure it however you like.

---

## Environment facts (not decisions — known constraints)

- **Hardware:** NVIDIA DGX Spark, GB10 Grace–Blackwell, **aarch64/ARM64**, unified memory, CUDA.
  - Some Python wheels are absent from PyPI for aarch64. Use the NVIDIA package index or build from source as needed. This is a known environment quirk, not an error.
- **Scale:** Data fits in memory. Single GPU. A passing run completes in **minutes**, not hours. A run trending toward many hours at toy scale indicates a problem — investigate before continuing.
- **Data source:** Alanine dipeptide trajectory from `mdshare`. The implementation decides preprocessing and splits.
