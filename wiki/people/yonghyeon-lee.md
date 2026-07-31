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
- [[motion-manifold-flow-primitives-task-conditioned]] — MMFP (RA-L 2025): decoupled unconditional motion manifold + conditional latent flow matching for task/language-conditioned trajectory generation. No code released.
- [[differentiable-motion-manifold-primitives-reactive-motion]] — sole-author, ICRA 2026 (accepted): DMMP, continuous-time differentiable motion manifolds + Trajectory Manifold Optimization for kinodynamic-constraint satisfaction; simulation-only 7-DoF throwing; code still "coming soon" (verified 2026-07-30).

## My notes

The single author node behind the motion-manifold/latent-space line. **Note the project's own posture has changed** (2026-07-25): that line is no longer the rope-swing project's chosen direction — it was demoted to one arm of a controlled shootout ([[sim-stage-b-amortization-shootout]], B4). See [[motion-manifold-primitives]] for the reasoning and the three confirmations that came out of ingesting this lineage.

**Attribution correction (2026-07-30): DA-MMP is NOT part of this author line.** [[da-mmp-learning-coordinated-accurate-throwing]] is by Chi Chu & Huazhe Xu (Shanghai Qi Zhi / Tsinghua IIIS) — an independent group building on MMP++, not a continuation of it.

Code availability across the lineage (re-checked 2026-07-30): **MMP++** (`Gabe-YHLee/MMPpp-public`, PyTorch, no license stated — pin the commit) and **EMMP** (`dlsfldl/EMMP-public`, MIT) are the only working entry points. **DMMP: code "coming soon"** at <https://diffmmp.github.io/> — that page also says "Paper (Coming soon)", which is **stale**: the paper is on arXiv at <https://arxiv.org/abs/2410.12193> and is the source this wiki ingested. **MMFP: code "coming soon"** at <https://mmflowp.github.io/>. Both project pages are on the standing watch list — see the Watch list in `.agent/NOTES.md`. GitHub: `Gabe-YHLee`. KIAS → MIT (Sangbae Kim's Biomimetic Robotics Lab) — the lineage is drifting toward dynamic legged/dynamic skills, which is exactly the regime where a manifold-of-swings action space would live.

**EMMP is the highest-value unclaimed follow-up ingest in this lineage**: it is the discrete-time baseline both MMFP and DMMP benchmark against, and the only member with usable public code (MIT).
