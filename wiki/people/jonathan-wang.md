---
name: "Jonathan Wang"
affiliation: "AUTOLAB, UC Berkeley (at time of paper)"
tags: [DLO, robot-learning, sim-to-real, dynamic-manipulation, free-end-cable, autolab]
homepage: ""
scholar: ""
date_updated: 2026-05-06
---

## Research areas

- Dynamic manipulation of free-end cables and other 1D deformable objects
- Sim-to-real transfer via simulator tuning + supervised forward dynamics
- Self-supervised data collection pipelines for robot manipulation

## Key papers

- [[self-supervised-learning-dynamic-planar-manipulation]] (arXiv 2024) — co-first author. Introduces the four-parameter [[two-arc-planar-motion-primitive]] for dynamic planar free-end cable manipulation; extends [[planar-robot-casting-real2sim2real-self-supervised]] from the predecessor PRC pipeline to longer free-end cables; achieves 22–34% per-cable median tip error on three cables on a UR5.

## Recent work

- [[self-supervised-learning-dynamic-planar-manipulation]]
(no further wiki-ingested papers attributed to Jonathan Wang at this time)

## Collaborators

- Huang Huang (Berkeley AUTOLAB; co-first author on [[self-supervised-learning-dynamic-planar-manipulation]])
- [[vincent-lim]] (Berkeley AUTOLAB; first author of the immediate predecessor [[planar-robot-casting-real2sim2real-self-supervised]])
- Harry Zhang, Jeffrey Ichnowski, Daniel Seita, Yunliang Chen (Berkeley AUTOLAB)
- Ken Goldberg (Berkeley AUTOLAB advisor)

## My notes

Co-first author on the direct successor of [[planar-robot-casting-real2sim2real-self-supervised]]. The action-parameterization piece — adding the wrist rotation $\psi$ to convert a planar two-arc sweep into a 3D scoop and going from $A_1=3$ params to $A_2=4$ params — is the primary technical contribution distinct from the predecessor's recipe; under his name it shows up as the main mechanism that increases simulated workspace coverage from ~66% to ~80%.
