---
name: "Jeffrey Ichnowski"
affiliation: "Robotics Institute, Carnegie Mellon University (formerly UC Berkeley AUTOLab)"
tags: [robot-learning, deformable-object-manipulation, cable-manipulation, sim-to-real, AUTOLAB]
homepage: ""
scholar: ""
date_updated: "2026-05-06"
---

## Research areas

- [[dynamic-dlo-tip-targeting]]
- Deformable-object manipulation (cables, ropes, cloth) with robot learning
- Sim-to-real transfer for dynamic manipulation tasks
- Optimization-based and learning-based motion planning
- Berkeley AUTOLAB lineage of cable-casting and untangling research; carries that line forward at CMU

## Key papers

- [[planar-robot-casting-real2sim2real-self-supervised]] (ICRA 2022) — co-author. Real2Sim2Real pipeline with Differential-Evolution simulator tuning for planar cable casting.
- [[robots-lost-arc-self-supervised-learning]] (ICRA 2022) — co-author. Self-supervised learning of 3D apex-point arcing trajectories for fixed-endpoint cable manipulation.
- [[self-supervised-learning-dynamic-planar-manipulation]] (arXiv 2024) — co-author. Direct extension of Real2Sim2Real to free-end cable target reaching.
- [[wiggle-go-system-identification-zero-shot]] (arXiv 2026) — co-author. Decoupled wiggle-probe sysID + Drake CMA-ES trajectory optimization for zero-shot 3D rope-tip striking.

## Recent work

- [[robots-lost-arc-self-supervised-learning]]
- [[wiggle-go-system-identification-zero-shot]]
(Spans the Berkeley AUTOLAB → CMU transition; key papers in our wiki bracket 2022–2026.)

## Collaborators

- Ken Goldberg (Berkeley AUTOLAB advisor on Real2Sim2Real, Robots-of-the-Lost-Arc, Free-End-Cable)
- Krishna Suresh, Arthur Jakobsson (CMU Robotics Institute) — Wiggle-and-Go coauthors
- Harry Zhang, Vincent Lim, Daniel Seita, Jonathan Wang (Berkeley AUTOLAB)

## My notes

Ichnowski is a stable connective node across the Berkeley AUTOLAB and CMU Robotics Institute cable-manipulation threads — he co-authors three of the 2022 Berkeley papers in this wiki (Real2Sim2Real, Robots-of-the-Lost-Arc, Free-End-Cable) and the 2026 CMU Wiggle-and-Go paper. The shift from "tune a sim, learn a single-step regressor" (Berkeley line) to "wiggle-probe sysID, then trajectory optimization" (CMU line) is partly traceable through his coauthor history.
