---
title: "Sim-to-Real and Rapid Adaptation"
tags: [sim-to-real, rapid-adaptation, meta-learning, context-encoder, domain-randomization, system-identification, real2sim, online-adaptation, robot-learning]
key_venues: [RSS, CoRL, ICRA, IROS, ICML, NeurIPS]
related_topics:
  - "[[dynamic-dlo-tip-targeting]]"
  - "[[model-based-planning-for-manipulation]]"
  - "[[dynamic-deformable-object-simulation]]"
key_people:
  - "[[deepak-pathak]]"
  - "[[ashish-kumar]]"
  - "[[sergey-levine]]"
  - "[[jitendra-malik]]"
key_papers:
  - "[[rma-rapid-motor-adaptation-legged-robots]]"
  - "[[rapid-adaptation-particle-dynamics-generalized-deformable]]"
  - "[[learning-adapt-dynamic-real-world-environments]]"
  - "[[wiggle-go-system-identification-zero-shot]]"
  - "[[planar-robot-casting-real2sim2real-self-supervised]]"
  - "[[deform-differentiable-discrete-elastic-rods-real]]"
linked_ideas: []
---

## Overview

Transferring a dynamic policy trained in simulation to a real, never-before-seen rope is the linchpin of open-loop zero-shot tip targeting: the policy must absorb the unknown physical parameters of *this* rope (mass per length, bending/twisting stiffness, damping) with at most a few minutes of calibration, then commit to an open-loop swing. This topic collects the transfer toolbox — meta-learning, context-encoder adaptation, domain randomization, system identification, and real2sim pipelines. The canonical adaptation architecture is **[[rma-rapid-motor-adaptation-legged-robots]]** (RMA): a privileged-information teacher trained in sim, distilled into a student that infers a latent "extrinsics" vector from recent observation history — adapting online without explicit system-ID. **[[rapid-adaptation-particle-dynamics-generalized-deformable]]** ports this teacher-student recipe to deformable particle dynamics, and **[[learning-adapt-dynamic-real-world-environments]]** (GrBAL/ReBAL) does the model-based analog, meta-training a dynamics model that re-fits within a handful of timesteps. The explicit-system-ID line — **[[wiggle-go-system-identification-zero-shot]]** (one safe wiggle → parameter estimate → zero-shot strike) and **[[planar-robot-casting-real2sim2real-self-supervised]]** (differential-evolution real2sim tuning) — represents the complementary "calibrate-then-commit" strategy that best matches the few-minute-per-object target. **[[deform-differentiable-discrete-elastic-rods-real]]** supplies the differentiable simulator and neural residual that make per-object identification fast and accurate.

## Timeline

| Year | Anchor result | What changed |
|---|---|---|
| 2019 | GrBAL / ReBAL | Meta-learned dynamics models adapt online within a few timesteps of new real-world dynamics |
| 2021 | RMA | Two-phase teacher-student context-encoder adaptation enables zero-shot legged sim-to-real with online extrinsics inference |
| 2022 | Real2Sim2Real (planar casting) | Differential-evolution simulator tuning + supervised regression for per-cable dynamic-action transfer |
| 2024 | DEFORM | Differentiable discrete elastic rods + neural residual for fast, accurate per-DLO system identification |
| 2026 | Wiggle&Go / RAPiD | Single-wiggle task-agnostic system-ID for zero-shot 3D rope striking; RMA-style particle-dynamics adaptation for >80% real-world mobile-manipulation success |

## Seminal works

- **[[rma-rapid-motor-adaptation-legged-robots]]** (Kumar et al., RSS 2021) — defines [[amortized-context-encoder-adaptation]]: teacher with privileged dynamics, student inferring extrinsics from observation history for online adaptation.
- **[[learning-adapt-dynamic-real-world-environments]]** (Nagabandi et al., ICLR 2019) — meta-reinforcement-learning of a dynamics model ([[online-meta-learned-dynamics-adaptation]]) that adapts in real time under MPC.

## SOTA tracker

- **[[wiggle-go-system-identification-zero-shot]]** (CMU 2026) — [[task-agnostic-system-identification]] from one safe wiggle, enabling zero-shot real-hardware 3D rope striking.
- **[[rapid-adaptation-particle-dynamics-generalized-deformable]]** (UT-Austin + Stanford 2026) — [[rma-particle-dynamics-adaptation]] (privileged particle-position teacher → depth-policy student); >80% real-world success on 22-DOF mobile manipulation.
- **[[deform-differentiable-discrete-elastic-rods-real]]** (Chen et al., CoRL 2024) — [[implicit-system-identification]] via differentiable DER + neural residual on real industrial DLOs.

## Key benchmarks

- Per-cable real-hardware tip-error transfer ([[planar-robot-casting-real2sim2real-self-supervised]]) measured after differential-evolution simulator tuning.
- Zero-shot real-world success rate after a fixed calibration budget (Wiggle&Go single-wiggle; RAPiD adaptation).

## Open problems

### Known gaps

- **Fast (whip-class, >5 m/s tip) sim-to-real**: domain randomization alone has not been shown to close the gap at whip speeds; current real2sim and iterative-residual methods are validated mostly in slower regimes.
- **Calibration budget**: "a few minutes per object then zero-shot" is the target; methods that need many real attempts per goal fall short of it.

### Methodological gaps

- **Implicit vs explicit identification for whipping**: RMA-style amortized adaptation vs single-wiggle explicit system-ID have not been compared head-to-head on free-tip 3D targeting.
- **Identifiability of high-speed DLO parameters** from a short, safe calibration motion remains open.

## Concepts
- [[rma-particle-dynamics-adaptation]]
- [[task-agnostic-system-identification]]
- [[amortized-context-encoder-adaptation]]
- [[differential-evolution-sim-tuning]]
- [[implicit-system-identification]]
- [[real2sim2real-pipeline]]
