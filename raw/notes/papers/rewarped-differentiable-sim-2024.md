---
title: "Stabilizing Reinforcement Learning in Differentiable Multiphysics Simulation"
authors: ["Eliot Xing", "Vernon Luk", "Jean Oh"]
venue: "ICLR 2025 (Spotlight)"
year: 2025
arxiv_id: "2412.12089"
doi: ""
note_type: bibliography_only
sources: [field-research]
---

# Rewarped: Differentiable Multiphysics Sim + SAPO

**One-line gist**: A parallel differentiable multiphysics simulation platform (Rewarped) plus a maximum-entropy first-order model-based actor-critic algorithm (SAPO) that stably exploits analytic gradients through soft-body/contact dynamics.

**Task/Method setup**: Manipulation and locomotion tasks involving rigid-body + deformable material interaction. SAPO uses analytic policy gradients from differentiable sim, wrapped in a maximum-entropy actor-critic framework to regularize the notoriously unstable gradient signal through contact discontinuities.

**Sim vs real**: Primarily a sim-side contribution; the Rewarped platform supports massively parallel rollouts with GPU acceleration across rigid, articulated, and deformable bodies. No explicit sim2real transfer protocol is presented.

**Core idea / mechanism**:
- Rewarped exposes differentiable gradients through multiphysics (soft bodies, contact) in a batched GPU setting.
- SAPO = first-order analytic gradients (like SHAC/DiffRL) + entropy regularization to dampen gradient explosion near contact events.
- Critic provides a value baseline; stochastic actor training via reparameterization through the sim graph stabilizes learning without zeroth-order fallback.

**Why it matters for OUR problem**:
- **Forward model**: Rewarped is a candidate GPU-parallel differentiable simulator for rope/DLO dynamics — directly relevant to training a tip-trajectory forward model.
- **Robust planning**: SAPO's entropy-regularized analytic gradients offer a path to gradient-based trajectory optimization through our spline decoder without mode collapse on rope contact/slack events.
- **Compact action**: The differentiable sim + gradient flow is compatible with optimizing a compact min-jerk spline parameterization end-to-end.
- **Sim2real gap**: Platform does not address calibration/meta-adaptation; our RMA-style context encoder would need to sit on top.

**Key result**: SAPO outperforms model-free baselines (SAC, PPO) and prior differentiable-sim RL methods (SHAC) on tasks with rigid–deformable interaction; Rewarped achieves faster wall-clock training than sequential differentiable simulators.
