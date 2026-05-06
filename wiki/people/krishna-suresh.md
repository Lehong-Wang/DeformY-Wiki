---
name: "Krishna Suresh"
affiliation: "Carnegie Mellon University"
tags: [robot-learning, iterative-learning-control, ILC, deformable-object-manipulation, rope-manipulation]
homepage: ""
scholar: ""
date_updated: "2026-05-06"
---

## Research areas

- Iterative Learning Control for real-world robot manipulation
- Dynamic deformable-object manipulation (rope, cable, knot tying)
- Sample-efficient real-system learning (ILC instead of large-scale data collection)
- Optimization-based control (QP / norm-optimal ILC formulations)

## Key papers

- [[learning-deformable-object-manipulation-using-task]] (2026, with Chris Atkeson) — introduces Task-Level ILC with a critical-point objective for dynamic rope manipulation; achieves 100% flying-knot success in under 10 real trials across 7 rope types on xArm 7.

## Recent work

(Track: Suresh's S2 record shows ~6 papers as of 2026-05-06; this Task-Level ILC paper is the most-cited line.)

## Collaborators

- [[chris-atkeson]] (CMU, advisor) — co-author on the Task-Level ILC paper.

## My notes

Suresh is the first-author lead on the Task-Level ILC for deformable-object manipulation work. The paper's main intellectual move — combining critical-point objectives with task-level (object-state) error rather than robot-state error — is consistent with a research line that takes seriously how little simulation it takes to learn a hard task on real hardware, given the right cost shaping. Worth tracking what comes next, especially auto-discovery of critical points and extensions beyond rope.
