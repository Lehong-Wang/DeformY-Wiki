---
title: "Compact Action Parameterization"
tags: [action-representation, low-dimensional-action, motion-primitive, apex-point, minimum-jerk, trajectory-optimization, dynamic-manipulation, data-efficiency]
key_venues: [ICRA, IROS, RSS, CoRL, RA-L]
related_topics:
  - "[[dynamic-dlo-tip-targeting]]"
  - "[[dynamic-throwing-and-hitting]]"
  - "[[model-based-planning-for-manipulation]]"
key_people:
  - "[[harry-zhang]]"
  - "[[ken-goldberg]]"
  - "[[yonghyeon-lee]]"
  - "[[frank-park]]"
key_papers:
  - "[[robots-lost-arc-self-supervised-learning]]"
  - "[[planar-robot-casting-real2sim2real-self-supervised]]"
  - "[[self-supervised-learning-dynamic-planar-manipulation]]"
  - "[[mmp-motion-manifold-primitives-parametric-curve]]"
  - "[[motion-manifold-flow-primitives-task-conditioned]]"
  - "[[differentiable-motion-manifold-primitives-reactive-motion]]"
  - "[[diffusion-policy-visuomotor-policy-learning-action]]"
linked_ideas:
  - "[[direction-conditioned-open-loop-rope-tip-targeting]]"
---

## Overview

A low-dimensional, smooth action space is the practical key to data-efficient and exploitation-resistant dynamic skill learning: instead of regressing or planning over a full high-frequency joint trajectory, the policy emits a handful of parameters that a fixed planner expands into a dynamically feasible motion. This compresses the learning target, guarantees smoothness and feasibility by construction, and — crucially for the research direction — denies an offline planner the degrees of freedom it would need to find degenerate, model-exploiting trajectories. The canonical instances are **[[robots-lost-arc-self-supervised-learning]]** ([[apex-point-trajectory-parameterization]]: a single 3D waypoint plus a minimum-jerk QP fills in the rest of the motion), the casting primitives of **[[planar-robot-casting-real2sim2real-self-supervised]]** and **[[self-supervised-learning-dynamic-planar-manipulation]]** ([[two-arc-planar-motion-primitive]]), and the inverse-model view of trajectory generation ([[optimization-based-inverse-model]]). All three reduce a dynamic cable-manipulation action to a few parameters, collapsing real-world data demand to hundreds of trials while keeping the motion fast and feasible. The throwing line ([[dynamic-throwing-and-hitting]]) supplies parallel primitives (e.g. throwing motion primitives), and the same compact representations are the natural action interface for the forward-model-plus-planner formulation in [[model-based-planning-for-manipulation]].

## Timeline

| Year | Anchor result | What changed |
|---|---|---|
| 2022 | Lost-Arc / Real2Sim2Real | 3D apex-point parameterization and the two-arc casting primitive establish compact action spaces for dynamic cable manipulation |
| 2024 | Free-end-cable casting | The two-arc planar primitive is extended from fixed-endpoint to free-end cables |

## Seminal works

- **[[robots-lost-arc-self-supervised-learning]]** (Zhang et al., ICRA 2022) — introduces [[apex-point-trajectory-parameterization]]: regress a single 3D apex, let a minimum-jerk QP generate the fastest feasible trajectory through it.
- **[[planar-robot-casting-real2sim2real-self-supervised]]** (Lim et al., ICRA 2022) — parameterized two-arc casting primitive on UR5; per-cable median tip error of 8–14% of cable length on planar casting beyond the workspace.
- **[[mmp-motion-manifold-primitives-parametric-curve]]** (Lee, T-RO 2024, MMP++/IMMP++) — the *learned* end of the compact-action spectrum: autoencode demonstrations' parametric-curve parameters into a 2–5-dim latent manifold ([[motion-manifold-primitives]]) with a density-model feasibility prior; via-point/temporal modulation and millisecond latent-space replanning (0.006–0.077 s vs 2–205 s RRT-Connect).
- **[[motion-manifold-flow-primitives-task-conditioned]]** (Lee et al., RA-L 2025, MMFP) — moves task conditioning *out* of the decoder: an **unconditional** trajectory manifold plus a conditional flow-matching ODE over its 3-D latent, which is what lets it track [[complex-task-motion-dependencies]] where shared-prior conditional autoencoders collapse (joint accuracy 99.9% vs MMP 9.3%, TC-VAE 15.3%).
- **[[diffusion-policy-visuomotor-policy-learning-action]]** (Chi et al., RSS 2023 / IJRR 2024) — the *un*-compact end of the spectrum, included as the boundary case: raw per-timestep action chunks (predict $T_p$, execute $T_a \approx 8$) with no basis and no smoothness guarantee, made to work by an expressive score-based head. Establishes the action-horizon tradeoff that every compact parameterization implicitly resolves by setting $T_a = T_p$.
- **[[differentiable-motion-manifold-primitives-reactive-motion]]** (Lee, ICRA 2026, DMMP) — learned time bases instead of fixed ones: a decoder $f(z,t)$ built from ~100 learned basis functions. Notable for the *negative* half of its result — the architecture alone is **worse** than the discrete-time baseline (17.5% vs 77.4% success), and all the gain comes from fine-tuning against differentiable kinodynamic constraints ([[trajectory-manifold-optimization]]).

## SOTA tracker

- **[[self-supervised-learning-dynamic-planar-manipulation]]** (Wang et al. 2024) — extends the [[two-arc-planar-motion-primitive]] to free-end cables; 22–34% tip error across three real cables.
- **[[da-mmp-learning-coordinated-accurate-throwing]]** (Chu & Xu, 2025) — variable-length parameterization: the execution horizon L becomes an explicit component of the parameter vector (time-normalizing would re-time the motion and change release velocity). Gated-RBF + Hermite curves beat 32 uniform waypoints at matched parameter count by ~2× smoothness (MSSD 280.7/282.4/296.3 vs 596.4/558.9/555.9 at 0.9k/9k/90k trajectories) — **a gap that does not close with more data.**

## Key benchmarks

- Per-cable median tip error as a percentage of cable length on planar casting / free-end casting ([[planar-robot-casting-real2sim2real-self-supervised]], [[self-supervised-learning-dynamic-planar-manipulation]]).

## Open problems

### Known gaps

- **Single-waypoint expressivity ceiling**: a single apex cannot express multi-phase whip motions or very long cables; when to add a second waypoint is unresolved.
- **Open-loop within an episode**: compact-primitive actions are typically open-loop once executed, with no in-motion correction.
- **Planar-canonical parameterizations cannot shape out-of-plane arrival direction.** In-plane swings produce mostly in-plane arrival directions, so the whole low-dimensional planar family ([[apex-point-trajectory-parameterization]], [[two-arc-planar-motion-primitive]]) is expressively insufficient the moment the goal includes a direction. The open question is what the *minimum* dimension is that can cover arrival directions — [[smooth-basis-swing-parameterization]] uses ~25–37 as a working answer and ablates a physics-informed strike-frame family against it.
- **"Compact" and "smooth" are separable, and the literature conflates them.** A parametric-curve layer gives *hard* smoothness for every parameter vector including random ones; a learned latent over such curves ([[motion-manifold-primitives]]) adds only *statistical* smoothness — staying near the demonstration distribution. Which of the two a given result depends on is usually not reported. [[da-mmp-learning-coordinated-accurate-throwing]] separates them empirically: the curve family supplies structural C² smoothness (the MSSD gap is invariant to data scale), while its autoencoder ablation shows the *manifold* supplies executability that no amount of raw-parameter data supplies.
- **Latent dimension is unreported and unswept, everywhere.** MMFP uses m = 3 for ~15 discrete task labels, DMMP m = 32 with no ablation, DA-MMP m = 64 over 90k trajectories. No paper in the line reports success as a function of latent dimension, so "the manifold is low-dimensional" is an aesthetic claim rather than a measured one — and the number clearly grows with the conditioning space.

### Methodological gaps

- **Conditioning the parameterization on physical parameters** (cable length, mass per length) for cross-object generalization without retraining.
- **Exploitation-resistant planning**: characterizing how a compact action space bounds an offline planner's ability to exploit forward-model errors (link to [[model-based-planning-for-manipulation]]).

## Concepts
- [[optimization-based-inverse-model]]
- [[apex-point-trajectory-parameterization]]
- [[two-arc-planar-motion-primitive]]
- [[motion-manifold-primitives]]
- [[trajectory-manifold-optimization]] — amortizing constraint and task satisfaction into the generator by fine-tuning it against differentiable constraints over the whole conditioning space.
- [[complex-task-motion-dependencies]] — when conditioning specificity changes the mode count and support volume of the motion distribution, a shared-latent-prior conditional decoder cannot track it.
- [[receding-horizon-action-chunk-execution]] — the raw-timestep end of the spectrum: predict $T_p$, execute $T_a$, re-infer; $T_a$ is the single dial from reactive control to open-loop trajectory generation.
