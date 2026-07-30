---
name: "Yonghyeon Lee"
affiliation: "MIT (Biomimetic Robotics Lab, postdoc, since 2025); previously KIAS AI Research Fellow (2023-2025); PhD SNU Robotics Lab (Frank C. Park, 2023)"
research_areas: [motion-manifolds, movement-primitives, geometric-machine-learning, representation-learning, robot-learning, trajectory-generation]
homepage: "https://www.gabe-yhlee.com/"
scholar: "https://www.semanticscholar.org/author/3426462"
date_updated: 2026-07-23
type:
  kind: researcher
---

## Research areas

- [[compact-action-parameterization]]
Geometric machine learning for robot motion: the consistent thesis is that trajectory-generation problems should be solved on *low-dimensional, geometry-faithful latent manifolds* of motions. Originator and main driver of the Motion Manifold Primitives line — EMMP (equivariant MMP, CoRL 2023, co-first author), sole-author MMP++/IMMP++ ([[mmp-motion-manifold-primitives-parametric-curve]], T-RO 2024), motion manifold *flow* primitives (RA-L 2025, language/task-conditioned), and DMMP (differentiable MMP under kinodynamic constraints, ICRA 2026). Parallel threads: isometric/regularized autoencoder representation learning (the distortion-measure machinery IMMP++ reuses) and OSMP (Hopf limit-cycle motion policies).

## Recent work

- [[mmp-motion-manifold-primitives-parametric-curve]] — sole-author T-RO 2024: parametric-curve motion manifold primitives (MMP++) + isometric latent regularization (IMMP++), with public code, datasets, and pretrained models.

## My notes

The single author node behind the rope-swing project's chosen direction (motion-manifold/latent-space planning). Code availability across his lineage (checked 2026-07): **MMP++** (`Gabe-YHLee/MMPpp-public`, PyTorch, no license stated — pin the commit) and **EMMP** (`dlsfldl/EMMP-public`, MIT) are the working entry points; DMMP (ICRA 2026) and DA-MMP say "coming soon"; re-check periodically. GitHub: `Gabe-YHLee`. KIAS → MIT (Sangbae Kim's Biomimetic Robotics Lab) — the lineage is drifting toward dynamic legged/dynamic skills, which is exactly the regime where a manifold-of-swings action space would live.
