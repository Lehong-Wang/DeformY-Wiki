---
name: "Krishna Suresh"
<<<<<<< HEAD
affiliation: "Carnegie Mellon University"
tags: [robot-learning, iterative-learning-control, ILC, deformable-object-manipulation, rope-manipulation]
homepage: ""
scholar: ""
date_updated: "2026-05-06"
=======
affiliation: "Robotics Institute, Carnegie Mellon University"
tags: [DLO, robot-learning, dynamic-manipulation, rope-manipulation, iterative-learning-control, system-identification]
homepage: ""
scholar: ""
date_updated: 2026-05-06
>>>>>>> worktree-agent-a387b299d3acb5505
---

## Research areas

<<<<<<< HEAD
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
=======
- Dynamic manipulation of deformable linear objects (ropes, cables, knots) on real hardware
- Iterative learning control and system-identification approaches for sim-to-real rope dynamics
- CMU Robotics Institute lineage of dynamic-rope work that bridges learned residual policies and decoupled identification + trajectory optimization

## Key papers

- [[wiggle-go-system-identification-zero-shot]] (arXiv 2026, preprint) — co-author. Decoupled wiggle-probe sysID + Drake CMA-ES trajectory optimization pipeline for zero-shot 3D rope-tip striking on the xArm 7.
- Flying-Knot-ILC — co-author (this paper is being ingested in the same `/init` run; see sibling worktree). Iterative-learning-control approach to dynamic rope/knot manipulation. Shares CMU Robotics Institute lineage with Wiggle-and-Go.

## Recent work

(no further wiki-ingested papers attributed to Krishna Suresh at this time)

## Collaborators

- Arthur Jakobsson, Abhinav Mahajan, Karthik Pullalarevu, Yuemin Mao, Bardienus Duisterhof, Shahram Najam Syed, Jeffrey Ichnowski (CMU Robotics Institute) — Wiggle-and-Go coauthors
- Yunchao Yao (UNC Chapel Hill) — Wiggle-and-Go coauthor
- Flying-Knot-ILC coauthorship (CMU Robotics Institute lineage) — see sibling ingest

## My notes

Suresh is the connective tissue across the **CMU Robotics Institute** dynamic-rope-manipulation thread: he appears on Wiggle-and-Go (decoupled sysID + trajectory optimization, single-shot) and on Flying-Knot-ILC (iterative learning control, multi-attempt). The two papers are arguably the two ends of a contemporary debate — single-shot sysID-then-plan vs. iterative residual / ILC — and Suresh's authorship on both makes the institutional viewpoint visible. When the comparative real-hardware benchmark between the two design philosophies is eventually run, this is the lab and the person to watch.
>>>>>>> worktree-agent-a387b299d3acb5505
