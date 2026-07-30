---
title: "Motion Manifold Primitives"
aliases: ["MMP", "motion manifold", "manifold of trajectories", "latent trajectory manifold", "trajectory manifold autoencoder", "motion manifold hypothesis"]
tags: [movement-primitives, motion-manifold, autoencoder, latent-space-planning, learning-from-demonstration, trajectory-generation, action-representation, multi-modality]
maturity: active
definition: "A movement-primitive representation that autoencodes a set of demonstration trajectories for one task into a low-dimensional latent manifold with a density model on top, so that generating, modulating, and (re)planning task-completing trajectories all become operations on latent coordinates."
key_papers:
- "[[mmp-motion-manifold-primitives-parametric-curve]]"
first_introduced: "2020"
date_updated: 2026-07-23
related_concepts:
- "[[apex-point-trajectory-parameterization]]"
- "[[two-arc-planar-motion-primitive]]"
- "[[planning-as-diffusion]]"
parent_topic: compact-action-parameterization
---

## Definition

**Motion Manifold Primitives (MMP)** encode a motion skill not as one trajectory but as a *continuous manifold of diverse trajectories* that all complete the same task. Adopting the motion-manifold hypothesis — high-dimensional trajectory data for a task lies on a low-dimensional manifold — an autoencoder $g:\text{traj}\to\mathcal{Z}$, $f:\mathcal{Z}\to\text{traj}$ is trained on demonstrations, and a density model $p(z)$ (GMM, KDE, or a generative model) is fitted over the encoded latents. The decoder image is the motion manifold; the latent space plus density is a compact, sampleable parameterization of "all the ways to do this task". Task adaptation (avoid a new obstacle, hit a new goal) becomes a search or constrained optimization over the few latent coordinates, with an in-distribution constraint ($\log p(z)\ge\epsilon$) keeping solutions on the manifold of feasible behavior.

## Intuition

Classical movement primitives (DMP/ProMP/VMP) compress a skill to one parametric trajectory — easy to modulate, but brittle: if that trajectory becomes infeasible, the primitive has nothing else to offer. The manifold view keeps the *entire family* of demonstrated solutions alive in a space small enough (2–5 dims) that density estimation, sampling, and online optimization are all cheap. It is the learned generalization of hand-designed compact action spaces: instead of a human choosing "one apex point" or "two arcs" as the low-dimensional coordinates, the autoencoder discovers the coordinates from data, and the density model supplies the feasibility prior that hand-designed parameterizations get from their construction.

## Variants

- **Discrete-time MMP** (Noseworthy et al. 2020 TC-VAE; Lee et al. EMMP, CoRL 2023): decoder outputs a fixed-length sequence $(q_1,\dots,q_T)$; task-conditioned and equivariant versions exist. No temporal or via-point modulation.
- **MMP++** ([[mmp-motion-manifold-primitives-parametric-curve]]): decoder outputs *parametric-curve parameters* $w$ (ProMP/VMP-style affine models), restoring temporal modulation, hard via-point constraints, and smooth start/goal modulation while keeping the manifold.
- **IMMP++** (same paper): adds isometric regularization under a CurveGeom Riemannian metric (pullback of trajectory-space geometry onto curve parameters) so the latent space preserves the manifold's geometry — without it, distorted latents mis-cluster multi-modal demonstrations and the density model generates task-violating motions.
- **SE(3)/Lie-group MMP++**: orientation curves via geodesic interpolation times exponential-coordinate shape modulation; extends the manifold to pose trajectories.
- **Downstream line** (same author line): DMMP (differentiable MMP under kinodynamic constraints, ICRA 2026), DA-MMP, motion manifold *flow* primitives (latent flow models for language/task conditioning) — progressively swapping the latent density and adding conditioning.

## Comparison

- vs **hand-designed compact action spaces** ([[apex-point-trajectory-parameterization]], [[two-arc-planar-motion-primitive]]): same goal — a low-dimensional, feasible-by-construction action space — but the coordinates are learned from demonstrations rather than engineered, so expressivity is bounded by demonstration diversity rather than by a designer's template; the price is needing demonstrations and losing interpretability of the coordinates.
- vs **[[planning-as-diffusion]]**: both plan by generating from a learned model of the trajectory distribution and both keep plans "on-manifold" to resist degenerate solutions; diffusion models the distribution implicitly via denoising and steers with guidance gradients, while MMP models it explicitly as (decoder manifold + latent density) and steers by constrained optimization over latent coordinates — orders-of-magnitude cheaper at plan time, but restricted to a single task family per model and without diffusion's compositional guidance.
- vs **classical movement primitives** (DMP/ProMP/VMP): those encode one trajectory (or a Gaussian around one); MMP encodes a multi-modal family and makes diversity a first-class object.

## Known limitations

- Manifold coverage = demonstration coverage: nothing outside the demonstrated family can be generated, so demonstration diversity is the binding resource.
- Standard versions are not perception-conditioned — one model per environment/task instance; vision-conditioned manifolds are the active frontier (DMMP/DA-MMP line).
- Latent geometry is not free: without isometric or equivariance regularization, autoencoder latents distort distances and break density fitting (the core IMMP++ finding).
- Density-model choice leaks into behavior: GMMs fail on connected 1-D manifolds of demos (KDE needed), and rejection thresholds add hyperparameters.

## Open problems

- Isometric regularization for Lie-group-valued curve parameters (SO(3)/SE(3)) — open per [[mmp-motion-manifold-primitives-parametric-curve]].
- Conditioning the manifold on perception (arbitrary scene geometry) without exploding data requirements.
- Using MMP latents as an *action space for RL or model-based planning* on dynamic skills (throwing, rope swinging) where feasibility is dynamic, not just kinematic — the rope-swing project's intended use.
- Principled latent dimension selection vs the risk of decoder Jacobian rank drop at higher dims.

## Relationship to foundations

Extends [[movement-primitives]] from single-trajectory encodings to trajectory-manifold encodings, inheriting parametric-curve machinery (ProMP/VMP bases, via-point constraints) in the MMP++ variant. Operates in the [[imitation-learning]] (learning-from-demonstration) problem setting, and turns adaptation into small-scale [[trajectory-optimization]] over latent coordinates rather than over full trajectories.

## Realized by

- [[mmp-parametric-curve-motion-manifold-primitives]] — MMP++/IMMP++: affine-curve autoencoder + latent density (GMM/KDE) + isometric regularization + online iterative latent replanning (the wiki's reference realization, with public code).

## My understanding

For the rope-swing project this concept is the chosen middle path between hand-designed primitives (too rigid for 3D position+direction targets) and full-trajectory generative planners (too expensive, data-hungry). The plan-shaped question it answers: *what is the action space?* — a latent coordinate over a learned manifold of feasible swing trajectories, with the density model as a feasibility prior and via-point/temporal modulation as the task-conditioning knobs. The two transferable lessons from the founding line: (1) autoencode a *parametric-curve* representation, not raw discrete trajectories, or you lose every modulation capability that makes the primitive usable; (2) regularize latent geometry (isometry) or multi-modal families of swings will mis-cluster and the sampled swings will violate the task.
