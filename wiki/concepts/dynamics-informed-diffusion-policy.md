---
title: "Dynamics-Informed Diffusion Policy (DIDP)"
aliases: ["DIDP", "dynamics-informed diffusion policy", "physics-informed diffusion policy for DLO"]
tags: [DLO, diffusion-policy, imitation-learning, trajectory-optimization, reduced-order-model, robot-learning]
maturity: emerging
key_papers: ["[[dynamic-manipulation-deformable-objects-3d-simulation]]"]
first_introduced: "2025"
date_updated: 2026-05-06
related_concepts: ["[[physics-informed-test-time-adaptation]]", "[[reduced-order-gvs-model]]"]
---

## Definition

A **Dynamics-Informed Diffusion Policy (DIDP)** is a goal-conditioned diffusion policy whose action space is the **full reduced-order state of a coupled robot + deformable system** (rather than just the actuated joints), trained with a hybrid Imitation-Learning + Trajectory-Optimization (IL+TO) objective and deployed with a differentiable physics-guided test-time adaptation step. The defining commitments are:

1. The diffusion policy predicts the **whole-system trajectory** $\bm{Q} \in \mathbb{R}^{N\times D}$, including under-actuated DLO degrees of freedom — not just the controllable arm joints.
2. Pretraining stacks two losses: behavior cloning on expert demonstrations ($\mathcal{L}_{\text{diff}}$, with second-order angular-velocity matching) followed by a goal-directed trajectory-optimization fine-tune.
3. At sampling time, a [[physics-informed-test-time-adaptation]] mechanism injects a differentiable physics loss into the score of the reverse process to steer trajectories toward the goal in Cartesian space.

## Intuition

Plain goal-conditioned diffusion policies trained only on actuated-joint trajectories are essentially **action-imitation models**: they reproduce expert joint commands but have no internal model of how those commands move the rope tip. For dynamic DLO tasks the rope is the actual control variable, and small action perturbations cause large tip displacements. DIDP fixes this by (i) putting the rope state inside the policy so the model sees the rope dynamics it must steer, and (ii) using physics gradients at sampling time to nudge sampled trajectories toward the goal in Cartesian space without retraining the network.

## Formal notation

Let $D$ be the total reduced-order DoF (e.g. 20: 2 grid-arm + 18 rope) and $N$ the trajectory horizon. Forward diffusion: $\bm{Q}_t = \alpha_t \bm{Q}_0 + \sigma_t \epsilon$. Reverse model predicts the clean trajectory $\bm{Q}^\eta_{0|t}$. Pretraining loss
$$\mathcal{L}_{\text{diff}} = \lambda_Q \|\bm{Q} - \bm{Q}^\eta_{0|t}\|_2^2 + \lambda_{Q_d} \|\bm{Q}_d - \dot{\bm{Q}}^\eta_{0|t}\|_2^2$$
matches both position and angular velocity. The IL+TO finetune adds a differentiable goal loss $\mathcal{L}_{\text{pos}} = \|\log(\tilde{\mathbf{g}}_N^{-1} \cdot \mathbf{g}^\eta_N)\|^2$ on $SE(3)$, where $\mathbf{g}_N$ is the rope-tip pose computed from $\bm{Q}^\eta_{0|t}$ via the differentiable forward kinematics of the underlying [[reduced-order-gvs-model]]. At inference, the posterior score becomes
$$\nabla_{\bm{Q}_t}\log p(\bm{Q}_t|\bm{p}) \approx \nabla_{\bm{Q}_t}\log p(\bm{Q}_t) + \frac{\partial \mathcal{L}}{\partial \bm{Q}_{0|t}} \cdot \frac{\partial \bm{Q}_{0|t}}{\partial \bm{Q}_t}.$$

## Variants

- **Kinematics-only** ablation: predict only the 2 actuated joints, ignore the rope. Strictly weaker on dynamic DLO tasks (per DIDP ablation).
- **IL-only**: skip the trajectory-optimization fine-tune and the test-time adaptation. Useful as the imitation backbone but loses the goal-precision gains.
- **Full-finetune adaptation**: at inference, update *all* denoiser parameters instead of only the final projection layer. DIDP shows this overwrites the imitation prior and degrades performance.
- **TO-only**: skip imitation pretraining, use trajectory optimization from scratch — DIDP shows this collapses (3D dynamics are too high-dimensional to optimize without an IL prior).

## Comparison

- vs. **plain [[diffusion-policy]]** (Chi et al. 2023): plain diffusion policies operate on actuated-joint trajectories with goal conditioning but no full-system state and no physics-aware test-time guidance. DIDP differs on the action-space choice and the sampling-time guidance.
- vs. **iLQR / CHOMP trajectory optimization**: optimization-based methods are precise when models are accurate but brittle to modeling errors and expensive at run time. DIDP keeps an IL backbone for stability and uses physics only as a residual guidance signal.
- vs. **iterative-residual policies (IRP)** in 2D: IRP is restricted to planar tasks and depends on Jacobian-based residual feedback over image sequences. DIDP works in full 3D over a reduced-order state and does not assume task repeatability.
- vs. **end-to-end RL** (e.g. PPO with calibrated Cosserat sims): RL learns from interaction without demonstrations but is data-starved on high-DoF DLO tasks. DIDP leverages 55k expert trajectories and physics gradients instead of raw rollouts.

## When to use

- The task is **goal-conditioned dynamic DLO manipulation** (rope whipping, tip targeting, fast cable shaping) and the system has a usable reduced-order differentiable simulator.
- A reasonably large expert-trajectory dataset is available for IL pretraining.
- Tip / end-effector pose precision matters more than visuomotor closed-loop reactivity.

Skip when: real-time inference matters more than precision (TTA is expensive), the rope state is unobservable (DIDP needs full reduced-order state in the policy), or the goal is a topological configuration rather than a point in Cartesian space.

## Known limitations

- Sim-only validation in the originating paper; sim-to-real transfer untested.
- Single-rope, single-material training; generalization across DLO geometry/material is unstudied.
- TTA inflates inference cost (10s/sample with project-only finetuning) and breaks real-time guarantees.
- Demonstrated only on tip-position goals; orientation, multi-target, and sequential goals are not validated.

## Open problems

- How much of the gain comes from each component (full-state policy, IL+TO pretrain, PITA) when each is varied independently?
- Can DIDP be combined with consistency models / few-step samplers to bring inference under 1 s without losing PITA's physics correctness?
- Does the IL+TO + PITA recipe generalize to bimanual coordination and contact-rich DLO tasks (knotting, sewing)?

## Key papers

- [[dynamic-manipulation-deformable-objects-3d-simulation]] — introduces DIDP and the 3D rope-whipping benchmark; reports 84.3%@5cm and 20.8%@1cm in simulation.

## My understanding

DIDP is best read as a recipe rather than a single mechanism: **(IL on full-system reduced-order state) + (TO finetune) + (PITA at sampling time)**. The DIDP ablations show the three components contribute unevenly — the biggest single lift is moving from kinematics-only to full-state policy modeling, not the test-time adaptation. For DeformY's planned closed-loop sim-to-real DLO work, DIDP is the right benchmark baseline; the natural extension is to keep the recipe and evaluate it under closed-loop visual feedback and across multiple ropes/materials.
