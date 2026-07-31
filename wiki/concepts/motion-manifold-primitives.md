---
title: "Motion Manifold Primitives"
aliases: ["MMP", "motion manifold", "manifold of trajectories", "latent trajectory manifold", "trajectory manifold autoencoder", "motion manifold hypothesis"]
tags: [movement-primitives, motion-manifold, autoencoder, latent-space-planning, learning-from-demonstration, trajectory-generation, action-representation, multi-modality]
maturity: active
definition: "A movement-primitive representation that autoencodes a set of demonstration trajectories for one task into a low-dimensional latent manifold with a density model on top, so that generating, modulating, and (re)planning task-completing trajectories all become operations on latent coordinates."
key_papers:
- "[[mmp-motion-manifold-primitives-parametric-curve]]"
- "[[differentiable-motion-manifold-primitives-reactive-motion]]"
- "[[motion-manifold-flow-primitives-task-conditioned]]"
- "[[da-mmp-learning-coordinated-accurate-throwing]]"
first_introduced: "2020"
date_updated: 2026-07-30
related_concepts:
- "[[apex-point-trajectory-parameterization]]"
- "[[two-arc-planar-motion-primitive]]"
- "[[planning-as-diffusion]]"
- "[[trajectory-manifold-optimization]]"
- "[[complex-task-motion-dependencies]]"
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
- **MMFP** ([[motion-manifold-flow-primitives-task-conditioned]], Lee et al., RA-L 2025): decouples the two jobs — train an *unconditional* autoencoder over demonstrations, then fit a **conditional flow-matching vector field in its latent space**. This is what lets it track [[complex-task-motion-dependencies]], where a shared-latent-prior conditional autoencoder collapses (joint accuracy 99.9% vs MMP 9.3% / TC-VAE 15.3%) because conditioning shrinks the support.
- **DMMP / DMMFP** ([[differentiable-motion-manifold-primitives-reactive-motion]], Lee, ICRA 2026): the decoder takes time as an input, $f(z,t)$, via ~100 *learned* time bases in a DeepONet factorization, so exact $\dot q/\ddot q/\dddot q$ enter the training loss; adds [[trajectory-manifold-optimization]] to fine-tune the decoder against task + kinodynamic-constraint penalties across the whole task continuum. **Its own ablation is the important result**: the architecture alone scores 17.5% success, *worse* than the 77.4% discrete-time MMFP it replaces; TMO supplies all the gain (95.8%, 100% with rejection sampling).
- **DA-MMP** ([[da-mmp-learning-coordinated-accurate-throwing]], Chu & Xu 2025 — an **independent group**, *not* the Lee line): variable-length trajectories (execution horizon enters the parameter vector), a 90k planner-generated corpus instead of demonstrations, and a latent conditional flow-matching model conditioned on *executed* outcomes replacing the static latent density. 60% real ring-toss, above trained human experts at 56.7%.

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
- [[mmfp-motion-manifold-flow-primitives]] — MMFP: unconditional autoencoder manifold + conditional latent flow matching; the task/language-conditioned member of the line. No code released.
- [[dmmp-differentiable-motion-manifold-primitives]] — DMMP/DMMFP + TMO + rejection sampling: continuous-time decoder $f(z,t)$ with a DeepONet-style learned time basis, task-conditioned latent flow, and constraint-penalty fine-tuning of the decoder; 7-DoF dynamic throwing, latent dim 32, no public code.
- [[da-mmp-dynamics-aware-motion-manifold]] — DA-MMP: variable-length (duration-in-parameter) via-point RBF parameterization, AE manifold from 90k planner-generated trajectories, latent conditional flow matching on executed landing outcomes.

## My understanding

**Status correction (2026-07-30).** This section previously called MMP "the chosen middle path" for the rope-swing project. That was written 2026-07-23 and is **superseded**: the 2026-07-25 base-method decision demoted the manifold line from prerequisite to one arm of a controlled shootout ([[sim-stage-b-amortization-shootout]], B4), and the project's action space is now [[smooth-basis-swing-parameterization]] with [[conditional-flow-matching-motion-parameters]] over its raw parameters — no autoencoder.

The reasoning behind the demotion, now **corroborated from inside this literature**: MMP++'s smoothness lives in its *parametric-curve layer*, which the project already uses; the latent adds only statistical smoothness (staying near the demonstrated family). Three confirmations from the 2026-07-30 ingest:

1. **DMMP's data-collection stage is itself a fixed boundary-pinned Gaussian-basis curve model** — structurally the project's own action space — and it is used as the *ground truth* the neural manifold is trained to imitate. Its ablation shows the manifold architecture alone is worse than the discrete-time baseline; [[trajectory-manifold-optimization]] supplies all the gain.
2. **MMFP's anti-flow result does not transfer.** Its "flow in trajectory space fails, output is jerky" finding is against flow over raw ~5000-D discrete-time trajectories, not over compact smooth curve parameters — and its own conclusion names MMP++-style parametric curves as the alternative fix.
3. **Latent dimension does not stay small under continuous-goal conditioning.** MMFP uses m = 3 for ~15 discrete task labels; DMMP uses 32; DA-MMP uses 64 over 90k trajectories. Compression against a ~30-D parameter vector is therefore ~6×, not ~1000×, bought with a second training stage and a reconstruction bottleneck that caps diversity.

The two transferable lessons from the founding line still stand: (1) autoencode a *parametric-curve* representation, not raw discrete trajectories, or you lose every modulation capability that makes the primitive usable; (2) regularize latent geometry (isometry) or multi-modal families of swings will mis-cluster and the sampled swings will violate the task. **A third, added by MMFP:** never condition by sharing one latent prior across tasks — if B4 is run at all, run it in MMFP form (unconditional manifold + conditional latent flow), because the TC-VAE form collapses exactly when conditioning sharpens, which is the project's own H4.
