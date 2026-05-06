---
name: "Krishna Suresh"
affiliation: "Robotics Institute, Carnegie Mellon University"
tags: [DLO, robot-learning, dynamic-manipulation, rope-manipulation, iterative-learning-control, ILC, system-identification, deformable-object-manipulation]
homepage: ""
scholar: ""
date_updated: "2026-05-06"
---

## Research areas

- Dynamic manipulation of deformable linear objects (ropes, cables, knots) on real hardware
- Iterative Learning Control (Task-Level ILC; norm-optimal QP formulations) for sample-efficient real-system learning
- System-identification approaches for sim-to-real rope dynamics
- CMU Robotics Institute lineage of dynamic-rope work that bridges learned residual policies and decoupled identification + trajectory optimization

## Key papers

- [[learning-deformable-object-manipulation-using-task]] (2026, with Chris Atkeson) — first author. Introduces Task-Level ILC with a critical-point objective for dynamic rope manipulation; achieves 100% flying-knot success in under 10 real trials across 7 rope types on xArm 7.
- [[wiggle-go-system-identification-zero-shot]] (arXiv 2026, preprint) — co-author. Decoupled wiggle-probe sysID + Drake CMA-ES trajectory optimization pipeline for zero-shot 3D rope-tip striking on the xArm 7.

## Recent work

(Suresh's S2 record shows ~6 papers as of 2026-05-06; the Task-Level ILC and Wiggle-and-Go papers are the most-cited lines.)

## Collaborators

- [[chris-atkeson]] (CMU, advisor on Task-Level ILC)
- Arthur Jakobsson, Abhinav Mahajan, Karthik Pullalarevu, Yuemin Mao, Bardienus Duisterhof, Shahram Najam Syed, Jeffrey Ichnowski (CMU Robotics Institute) — Wiggle-and-Go coauthors
- Yunchao Yao (UNC Chapel Hill) — Wiggle-and-Go coauthor

## My notes

Suresh is the connective tissue across the **CMU Robotics Institute** dynamic-rope-manipulation thread: he appears on Task-Level ILC (iterative learning control, multi-attempt) and on Wiggle-and-Go (decoupled sysID + trajectory optimization, single-shot). The two papers are arguably the two ends of a contemporary debate — single-shot sysID-then-plan vs. iterative residual / ILC — and Suresh's authorship on both makes the institutional viewpoint visible. When a comparative real-hardware benchmark between these two design philosophies is eventually run, this is the lab and the person to watch. The Task-Level ILC paper's main intellectual move — combining critical-point objectives with task-level (object-state) error rather than robot-state error — is consistent with a research line that takes seriously how little simulation it takes to learn a hard task on real hardware, given the right cost shaping.
