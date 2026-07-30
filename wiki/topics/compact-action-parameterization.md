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
key_papers:
  - "[[robots-lost-arc-self-supervised-learning]]"
  - "[[planar-robot-casting-real2sim2real-self-supervised]]"
  - "[[self-supervised-learning-dynamic-planar-manipulation]]"
  - "[[mmp-motion-manifold-primitives-parametric-curve]]"
linked_ideas: []
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

## SOTA tracker

- **[[self-supervised-learning-dynamic-planar-manipulation]]** (Wang et al. 2024) — extends the [[two-arc-planar-motion-primitive]] to free-end cables; 22–34% tip error across three real cables.

## Key benchmarks

- Per-cable median tip error as a percentage of cable length on planar casting / free-end casting ([[planar-robot-casting-real2sim2real-self-supervised]], [[self-supervised-learning-dynamic-planar-manipulation]]).

## Open problems

### Known gaps

- **Single-waypoint expressivity ceiling**: a single apex cannot express multi-phase whip motions or very long cables; when to add a second waypoint is unresolved.
- **Open-loop within an episode**: compact-primitive actions are typically open-loop once executed, with no in-motion correction.

### Methodological gaps

- **Conditioning the parameterization on physical parameters** (cable length, mass per length) for cross-object generalization without retraining.
- **Exploitation-resistant planning**: characterizing how a compact action space bounds an offline planner's ability to exploit forward-model errors (link to [[model-based-planning-for-manipulation]]).

## Concepts
- [[optimization-based-inverse-model]]
- [[apex-point-trajectory-parameterization]]
- [[two-arc-planar-motion-primitive]]
- [[motion-manifold-primitives]]
