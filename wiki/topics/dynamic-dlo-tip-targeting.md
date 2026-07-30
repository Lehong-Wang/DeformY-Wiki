---
title: "Dynamic DLO Tip Targeting"
tags: [DLO, deformable-linear-object, dynamic-manipulation, goal-conditioned, rope-whipping, tip-targeting, open-loop, zero-shot, robot-learning]
key_venues: [RSS, ICRA, IROS, CoRL, IJRR, RA-L]
related_topics:
  - "[[model-based-planning-for-manipulation]]"
  - "[[sim-to-real-and-rapid-adaptation]]"
  - "[[dynamic-deformable-object-simulation]]"
  - "[[dynamic-throwing-and-hitting]]"
  - "[[compact-action-parameterization]]"
key_people:
  - "[[shuran-song]]"
  - "[[cheng-chi]]"
  - "[[ken-goldberg]]"
  - "[[jeffrey-ichnowski]]"
  - "[[krishna-suresh]]"
key_papers:
  - "[[iterative-residual-policy-goal-conditioned-dynamic]]"
  - "[[robots-lost-arc-self-supervised-learning]]"
  - "[[wiggle-go-system-identification-zero-shot]]"
  - "[[dynamic-manipulation-deformable-objects-3d-simulation]]"
  - "[[self-supervised-learning-dynamic-planar-manipulation]]"
  - "[[planar-robot-casting-real2sim2real-self-supervised]]"
  - "[[implicit-physics-aware-policy-dynamic-manipulation]]"
  - "[[learning-deformable-object-manipulation-using-task]]"
linked_ideas: []
---

## Overview

Goal-conditioned dynamic manipulation of ropes and other deformable linear objects (DLOs) — whipping, casting, or swinging a free tip so it reaches a designated spatial goal — is the central task of this wiki's research direction. The defining ambition is **open-loop, zero-shot tip targeting to a 3D position (and ultimately a position + direction)**: a single dynamic arm motion that drives the rope tip to a goal with no per-target iteration and no closed-loop visual feedback during the swing. The literature is anchored by **[[iterative-residual-policy-goal-conditioned-dynamic]]** (Chi et al., RSS 2022 Best Paper / IJRR 2024), which whips a rope to a planar 3D target via a learned delta-dynamics network plus sampling-based action refinement over a low-dimensional swing primitive. Subsequent work pushes along three axes: real-hardware zero-shot/one-shot methods (**[[wiggle-go-system-identification-zero-shot]]**, **[[implicit-physics-aware-policy-dynamic-manipulation]]**, **[[learning-deformable-object-manipulation-using-task]]**), the first dedicated 3D rope-whip benchmark with leaderboard numbers (**[[dynamic-manipulation-deformable-objects-3d-simulation]]**, DIDP), and the planar/fixed-endpoint casting line (**[[planar-robot-casting-real2sim2real-self-supervised]]**, **[[self-supervised-learning-dynamic-planar-manipulation]]**, **[[robots-lost-arc-self-supervised-learning]]**). The open gap — synthesized in [[dlo-dynamic-tip-targeting]] — is that no published method simultaneously delivers real-robot hardware, arbitrary 3D targets, a learned runtime policy, and free-space dynamic whipping; each property exists in isolation but their intersection is the niche this direction targets.

## Timeline

| Year | Anchor result | What changed |
|---|---|---|
| 2022 | IRP / Real2Sim2Real / Lost-Arc | Three converging recipes for goal-conditioned dynamic cable: iterative residual on a swing primitive (IRP), DE-tuned simulator + supervised regression (R2S2R), and a learned 3D apex point (Lost-Arc) |
| 2024 | Free-end-cable casting | Tip-targeting extends from fixed-endpoint to free-end cables via the two-arc planar primitive |
| 2025 | DIDP / IPA | First 3D rope-whip benchmark with a leaderboard; one-shot rope-as-tool transport of rigid payloads |
| 2026 | Wiggle&Go / Flying-Knot-ILC | Real-hardware zero-shot 3D rope striking via system-ID-then-trajectory-optimization; task-level ILC on a real flying-knot task |

## Seminal works

- **[[iterative-residual-policy-goal-conditioned-dynamic]]** (Chi RSS 2022 / IJRR 2024) — the anchor of the literature. Learns a [[delta-dynamics-network]] over a low-dimensional swing primitive and iteratively refines the action toward a 3D goal; zero-shot sim-to-real across multiple ropes and embodiments. RSS 2022 Best Paper.
- **[[robots-lost-arc-self-supervised-learning]]** (Zhang et al., ICRA 2022) — 3D apex-point representation for fixed-endpoint cable manipulation, one of the three canonical action representations for dynamic cable tasks.

## SOTA tracker

- **[[dynamic-manipulation-deformable-objects-3d-simulation]]** (DIDP, 2025) — first 3D rope-whip benchmark with leaderboard; reports 84.3% within 5 cm and 20.8% within 1 cm (sim-only).
- **[[wiggle-go-system-identification-zero-shot]]** (CMU 2026) — closest published match to the gap-thesis target task: zero-shot real-hardware 3D point striking on xArm 7 via [[task-agnostic-system-identification]] + trajectory optimization.
- **[[implicit-physics-aware-policy-dynamic-manipulation]]** (Wang & Qureshi 2025) — one-shot rope-as-tool 3D-target transport, 62.5% real-world success on UR5e.
- **[[learning-deformable-object-manipulation-using-task]]** (Suresh & Atkeson, CMU 2026) — task-level ILC on real xArm 7, 100% success in ≤10 trials across 7 rope/cable types.

## Key benchmarks

- **3D rope-whip benchmark** (introduced by [[dynamic-manipulation-deformable-objects-3d-simulation]]) — leaderboard reporting success rates at 5 cm and 1 cm tip-error thresholds; sub-1-cm remains wide open (DIDP at 20.8%).
- **Free-end / fixed-endpoint planar casting** ([[planar-robot-casting-real2sim2real-self-supervised]], [[self-supervised-learning-dynamic-planar-manipulation]]) — per-cable median tip error reported as a percentage of cable length.

## Open problems

### Known gaps

- **The four-way intersection is unfilled**: no published method simultaneously delivers {real-robot hardware, arbitrary 3D target, learned runtime policy, free-space dynamic whipping}. IRP is real-hw + learned but planar; DIDP is 3D + learned but sim-only; Wiggle-and-Go is real-hw + 3D but uses trajectory optimization rather than a learned policy; IPA is real-hw + 3D + learned but conditions on rigid-payload landing, not free-tip.
- **Direction conditioning**: nearly all methods target a 3D *position* only; conditioning the tip on a position **and** approach direction is essentially unaddressed.
- **Sub-1-cm tip accuracy at whip-class speeds** remains far from solved on the public benchmark.

### Methodological gaps

- **Open-loop without per-target iteration**: IRP and task-level ILC require several real attempts per goal; a true single-shot open-loop policy across goals is not yet demonstrated.
- **Closed-loop within an episode**: most parameterized-primitive methods are open-loop once the arm leaves the start pose, with no fast in-swing correction.

## Concepts
- [[iterative-residual-policy]]
- [[task-level-iterative-learning-control]]
- [[heterogeneous-soft-rigid-system]]
- [[critical-point-objective]]
- [[delta-dynamics-network]]
