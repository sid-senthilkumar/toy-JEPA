# BACKGROUND.md — LeWM / JEPA / SIGReg Primer

This is a compact orientation for the agent and the reviewer. It is not a textbook; read it for vocabulary and method context, not for implementation guidance.

---

## The JEPA idea

A Joint-Embedding Predictive Architecture (JEPA) learns by predicting in **latent space**, not in observation space. Given two views of a sequence — a context and a target — the model maps both to a shared latent, and the learning signal comes from predicting the target's latent from the context's latent. This sidesteps the need to reconstruct high-dimensional, high-entropy details (pixel noise, bond vibrations) that are irrelevant to the slow dynamics of interest.

In the world-model variant used here, the two views are simply **consecutive (or near-consecutive) frames** along a trajectory. The encoder maps each frame to a latent; the predictor maps the current latent to the next latent; the objective drives predicted and actual next latents together.

---

## The LeWM objective

LeWorldModel (Maes et al., 2026) formalizes this as a two-term loss:

```
L = L_pred + lambda * SIGReg(Z)
```

- **L_pred** is the latent prediction loss — typically MSE between the predicted next latent and the encoder's output for the actual next frame.
- **SIGReg(Z)** is the anti-collapse regularizer (described below). `lambda` is a scalar weight.

That is the entire objective. No reconstruction term, no contrastive term, no VICReg-style multi-term expansion.

---

## SIGReg — why it exists and what it does

**The collapse problem.** A JEPA-style objective has a degenerate solution: map every input to the same constant latent. Prediction loss goes to zero; nothing is learned. All practical JEPA systems need a mechanism to prevent this.

**Older heuristics.** Earlier systems used one or more of: an EMA (exponential moving average) target/momentum encoder, a stop-gradient on the target branch, or a frozen pretrained encoder. These work, but they add architectural complexity and detach from the core learning objective.

**SIGReg.** LeWM's contribution is a single regularizer that renders all of those heuristics unnecessary. SIGReg pushes the distribution of latents toward an **isotropic Gaussian** — a distribution with full rank and no preferred direction. If the latents collapse (all outputs identical, or onto a low-dimensional submanifold), the Gaussian distance becomes large, and the regularizer penalizes it.

Mechanically, SIGReg uses **random 1D projections** of the latent batch, applies the **Epps–Pulley test** for normality to each projection, and aggregates the deviations. This is justified by the **Cramér–Wold theorem**: a multivariate distribution is isotropic Gaussian if and only if every 1D projection is univariate Gaussian.

The result is stable training without an EMA encoder, without stop-gradients, and without a pretrained backbone. The encoder, predictor, and the regularizer all update together end-to-end.

---

## Why alanine dipeptide is the right toy

Alanine dipeptide (Ala-Dip, Ace-Ala-NHMe) is a 22-atom molecule in explicit or implicit solvent. Its conformational dynamics are:

- **Low-dimensional.** The slow degrees of freedom are fully described by two backbone dihedral angles, phi (φ) and psi (ψ). Everything else relaxes quickly.
- **Well-characterized.** The phi/psi landscape (the Ramachandran plot) is known analytically and experimentally. Two metastable basins dominate: **C7eq** (roughly phi ≈ −80°, psi ≈ +70°) and **C7ax** (roughly phi ≈ +70°, psi ≈ −60°).
- **Data available.** Standard trajectories are published via `mdshare` (Nüske et al. / HTMD project) — no generation required.
- **Verifiable ground truth.** Because phi and psi are known, we can check whether the latent recovered the right structure without supervision — a clean binary pass/fail.

This makes alanine dipeptide the canonical test case for slow-dynamics methods in molecular ML.

---

## References

- **Maes et al. 2026.** LeWorldModel (LeWM): JEPA-style world modeling with SIGReg. arXiv:2603.19312.
- **Balestriero & LeCun 2025.** LeJEPA and SIGReg: the Sketched Isotropic Gaussian Regularizer and its theoretical grounding.
- **Vander Meersche et al. 2024.** ATLAS: protein MD trajectory database. *Nucleic Acids Research.* (Context for the later phase of this project — not relevant to the toy stage.)
- **mdshare.** Standard alanine dipeptide trajectory data source. `http://mdshare.org` / PyEMMA `mdshare` package.
