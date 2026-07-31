---
name: "Frank C. Park"
affiliation: "Seoul National University, Robotics Laboratory (Department of Mechanical Engineering)"
research_areas: [robot-geometry, lie-group-methods, motion-manifolds, movement-primitives, geometric-machine-learning, robot-kinematics, representation-learning]
homepage: "https://sites.google.com/robotics.snu.ac.kr/fcp/"
scholar: "https://www.semanticscholar.org/author/2137679287"
date_updated: 2026-07-30
type:
  kind: researcher
---

## Research areas

- [[compact-action-parameterization]]

Geometric methods in robotics: Lie-group formulations of kinematics and dynamics, Riemannian geometry of configuration and latent spaces, and — the thread relevant to this wiki — the SNU Robotics Lab line that treats motion generation as learning a *geometry-faithful low-dimensional manifold of trajectories*. Senior author across the whole Motion Manifold Primitives lineage (EMMP, MMP++, MMFP, DMMP, DA-MMP) and of the regularized/isometric autoencoder machinery those papers reuse. Long-standing editorial and community role in robotics (IEEE T-RO editor-in-chief 2012-2015; IEEE RAS leadership).

## Recent work

- [[motion-manifold-flow-primitives-task-conditioned]] — MMFP (RA-L 2025): decoupled unconditional motion manifold + conditional latent flow matching for task-conditioned trajectory generation under support-collapsing conditioning.

## My notes

The lab node, not the individual, is what matters for the rope-swing project: SNU Robotics Lab under Park is where the entire motion-manifold-primitives line originates, and every member of that line the project cares about (MMP++, MMFP, DMMP, DA-MMP) carries his name as senior author. Practical consequence for tracking: releases from this group are announced on per-paper project pages rather than a single lab repo, and the recent ones (MMFP, DMMP, DA-MMP) all say "code coming soon" — only EMMP (`dlsfldl/EMMP-public`, MIT) and MMP++ (`Gabe-YHLee/MMPpp-public`, no license) are actually usable as of 2026-07. Also the source of the geometric-regularization toolbox (relaxed distortion measure, isometric autoencoders) that any manifold arm of the shootout would reuse.
