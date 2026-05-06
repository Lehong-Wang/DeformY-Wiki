---
name: "Arthur Jakobsson"
affiliation: "Robotics Institute, Carnegie Mellon University"
tags: [DLO, robot-learning, dynamic-manipulation, system-identification, rope-manipulation, sim-to-real]
homepage: ""
scholar: ""
date_updated: 2026-05-06
---

## Research areas

- Zero-shot dynamic manipulation of deformable linear objects (ropes, cables) on real hardware
- System identification for sim-to-real rope manipulation: neural-network parameter prediction from short safe probe actions
- Decoupled identification + trajectory optimization pipelines for goal-conditioned dynamic tasks
- 3D rope-tip striking, lobbing, and draping with the xArm 7 and Drake simulation

## Key papers

- [[wiggle-go-system-identification-zero-shot]] (arXiv 2026, preprint) — first author. Introduces the Wiggle and Go! framework: a single planar wiggle of the robot end-effector + a TCN-MLP network trained entirely in simulation predicts a 9-D rope-parameter vector that drives CMA-ES trajectory optimization in Drake for zero-shot 3D-target striking on the xArm 7, achieving 3.55 cm average accuracy across in-domain ropes.

## Recent work

(no further wiki-ingested papers attributed to Arthur Jakobsson at this time)

## Collaborators

- Abhinav Mahajan (CMU Robotics Institute) — Wiggle-and-Go co-author
- Karthik Pullalarevu (CMU Robotics Institute) — Wiggle-and-Go co-author
- Krishna Suresh (CMU Robotics Institute) — Wiggle-and-Go co-author; appears in the Flying-Knot-ILC line of work as well, linking the Wiggle-and-Go and Flying-Knot threads via shared CMU authorship
- Yunchao Yao (UNC Chapel Hill) — Wiggle-and-Go co-author
- Yuemin Mao, Bardienus Duisterhof, Shahram Najam Syed (CMU Robotics Institute) — Wiggle-and-Go co-authors
- Jeffrey Ichnowski (CMU Robotics Institute) — senior author; was previously at Berkeley AUTOLAB and is the lineage connection from the Real2Sim2Real / Planar Robot Casting work ([[planar-robot-casting-real2sim2real-self-supervised]]) to the CMU dynamic-rope work

## My notes

First author of the only confirmed real-hardware single-shot 3D rope-tip-striking paper as of April 2026 (Wiggle and Go!). The contribution is the *decoupling* of identification from task execution rather than a new neural architecture — the TCN-MLP $\Phi$ is straightforward; the design statement is that one short safe action plus simulator priors can substitute for either large real-world datasets (Lim et al.'s R2S2R) or end-to-end iterative residual learning (Chi et al.'s IRP). Worth tracking what comes out of this group next: GPU-accelerated trajectory optimization to drop the 25-min-per-task ceiling, or active-wiggle / online-adaptation extensions to fix the OOD-saturation chain failure, are the natural follow-ups the paper itself flags.
