---
name: "Krishna Suresh"
affiliation: "Robotics Institute, Carnegie Mellon University"
tags: [DLO, robot-learning, dynamic-manipulation, rope-manipulation, iterative-learning-control, system-identification]
homepage: ""
scholar: ""
date_updated: 2026-05-06
---

## Research areas

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
