---
title: "Physics-Informed Test-Time Adaptation (PITA) for Diffusion Sampling"
aliases: ["PITA", "physics-informed TTA", "physics-guided diffusion sampling", "differentiable-prior diffusion guidance"]
tags: [diffusion-policy, test-time-adaptation, score-based-sampling, physics-prior, robot-learning, DLO]
maturity: emerging
key_papers: ["[[dynamic-manipulation-deformable-objects-3d-simulation]]"]
first_introduced: "2025"
date_updated: 2026-05-06
related_concepts: ["[[dynamics-informed-diffusion-policy]]", "[[reduced-order-gvs-model]]"]
---

## Definition

**Physics-Informed Test-Time Adaptation (PITA)** is a sampling-time mechanism that augments a pretrained diffusion policy's reverse process with a **differentiable physics likelihood**, steering each denoising step toward physically consistent trajectories without retraining the network from scratch. Concretely, the posterior score is approximated as
$$\nabla_{\bm{Q}_t}\log p(\bm{Q}_t\mid \bm{p}) \approx \nabla_{\bm{Q}_t}\log p(\bm{Q}_t) + \nabla_{\bm{Q}_t}\log p(\bm{p}\mid \bm{Q}_t),$$
where $p(\bm{p}\mid \bm{Q}_{0|t}) \propto \exp(-\mathcal{L}(\bm{Q}_{0|t}))$ encodes a soft physics constraint. Two PITA losses operate in concert:

- a **Differential Dynamics Prior (DDP)** $\mathcal{L}_{\text{pos}}$ on $SE(3)$, propagating the goal-pose mismatch through a differentiable forward-kinematics model;
- a **Kinematic Boundary Condition (KBC)** $\mathcal{L}_{\text{KBC}}$ that pins the trajectory's start state to the rest configuration ($\bm{Q}(0)=\dot{\bm{Q}}(0)=\ddot{\bm{Q}}(0)=0$).

To preserve the imitation prior learned during pretraining, adaptation is restricted to the **final projection layer** of the denoiser network.

## Intuition

A pretrained diffusion policy can generate behaviorally plausible action sequences but has no built-in mechanism to satisfy strict physical constraints (start-state continuity, end-effector-at-goal). PITA treats the physics constraint as a likelihood and corrects the sampling trajectory by adding the gradient of that likelihood to the score. Restricting parameter updates to the final projection keeps the rest of the denoiser frozen, so the imitation prior is not erased by the test-time gradient. This is conceptually analogous to **classifier guidance** in image diffusion — but the "classifier" here is a physics simulator, not a learned discriminator.

## Formal notation

Given a pretrained denoiser $\epsilon_\eta$ and a differentiable physics loss $\mathcal{L}: \bm{Q}_{0|t} \mapsto \mathbb{R}_{\ge 0}$, the gradient added to each sampling step is
$$\nabla_{\bm{Q}_t}\log p(\bm{p}\mid \bm{Q}_t) = \frac{\partial \mathcal{L}}{\partial \bm{Q}_{0|t}} \cdot \frac{\partial \bm{Q}_{0|t}}{\partial \bm{Q}_t}.$$
The first factor requires the simulator be differentiable; the second is the standard Tweedie / DDIM Jacobian through the denoiser. In DIDP the physics loss is
$$\mathcal{L} = \mathcal{L}_{\text{pos}} + \lambda_{\text{KBC}} \mathcal{L}_{\text{KBC}},\quad \mathcal{L}_{\text{pos}} = \|\log(\tilde{\mathbf{g}}_N^{-1}\cdot \mathbf{g}^\eta_N)\|^2,$$
with the SE(3) right-Jacobian inverse appearing in the chain-rule expansion through the [[reduced-order-gvs-model]] recursion.

## Variants

- **DDP only (no KBC).** Adds the goal-pose loss only. DIDP shows this destabilizes trajectories — without KBC, the initial state drifts and the rope produces physically implausible motions.
- **KBC only (no DDP).** Anchors the start state but provides no goal-direction signal at sampling time; reduces to a regularized diffusion policy.
- **Full-network adaptation.** Update all denoiser parameters during PITA. DIDP shows this overwrites the imitation prior and degrades performance plus inflates inference cost ~40%.
- **Project-only adaptation** (the default in DIDP). Update only the final projection layer; preserves the IL prior; ~10s/sample inference.

## Comparison

- vs. **classifier-free guidance**: CFG modulates the conditional/unconditional score balance using a *learned* condition. PITA injects an *external, simulator-based* gradient. PITA can encode hard physical laws CFG cannot represent.
- vs. **Diffusion Posterior Sampling (DPS)** for inverse problems: DPS also adds $\nabla_{\bm{x}_t}\log p(\bm{y}\mid\bm{x}_t)$ where $\bm{y}=A\bm{x}_0+\eta$. PITA generalizes DPS to a non-linear differentiable physics operator (the GVS forward kinematics + dynamics) and adds kinematic boundary regularization.
- vs. **finetuning the entire diffusion policy** on goal data: full finetuning at inference time is what "Full Finetune" in DIDP is — DIDP shows project-only PITA outperforms it, suggesting that locality of the adaptation matters as much as the physics gradient itself.

## When to use

- The downstream task admits a **differentiable physics model** that can map predicted actions to a goal-relevant quantity (end-effector pose, contact force, energy).
- A pretrained imitation-style diffusion policy already produces structurally reasonable trajectories, and the remaining gap is precision / hard constraints.
- Inference latency budget allows several extra gradient steps per sample.

Skip when: the physics model is not differentiable, real-time inference is required, or the goal is purely behavioral (style imitation) without measurable constraints.

## Known limitations

- **Inference cost.** Project-only PITA still costs ~10s/sample in DIDP — too slow for closed-loop reactive control.
- **Both terms are needed.** DDP without KBC can move the trajectory's free start, producing physically impossible motions. The two losses are not interchangeable.
- **Adaptation locality is empirical.** Project-layer-only is the heuristic that worked for DIDP; the right adaptation surface for other architectures is unknown.
- **Differentiable simulator is a hard prerequisite.** Many existing physics simulators (rigid-body LCP solvers, generic FEM) are not end-to-end differentiable without substantial reimplementation.

## Open problems

- Stability theory: under what conditions does the score-plus-physics gradient remain bounded and produce a valid posterior, especially as the physics loss grows large?
- How does PITA interact with few-step samplers (consistency models, distillation)? Naively, fewer steps mean less guidance budget.
- Can PITA be used during *training* as a regularizer rather than only at test time, e.g. as a physics-grounded auxiliary loss?
- Multi-constraint PITA: combining DDP, KBC, and contact constraints for richer manipulation tasks.

## Key papers

- [[dynamic-manipulation-deformable-objects-3d-simulation]] — introduces PITA in the DIDP framework; reports that DDP+KBC together drive the @5cm success rate from 80.0% (no TTA) to 84.3% on 3D rope whipping, while DDP alone (without KBC) collapses it to 12.4%.

## My understanding

PITA is a clean instance of "hard physics constraints as test-time guidance for soft-prior generative models". The mechanism is general — any differentiable simulator + start-state regularizer should slot in — but the DIDP ablations carry an important warning: the boundary condition is not optional, and locality of the adaptation matters. For DeformY follow-on work, PITA is a candidate for any setting where the policy must respect SE(3) goals and rest-state continuity; the open engineering question is whether it can be made fast enough for closed-loop control.
