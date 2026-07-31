---
title: "DaXBench: Benchmarking Deformable Object Manipulation with Differentiable Physics"
slug: "daxbench-benchmarking-deformable-object-manipulation-differentiable"
arxiv: "2210.13066"
venue: "ICLR (Oral)"
year: 2023
tags: [DLO, deformable-object-manipulation, differentiable-physics, JAX, MPM, mass-spring, benchmark, RL, imitation-learning, planning, APG, SHAC, PPO, sim-to-real]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "fc715a1aac98fd5f5609e8fae16905ec3b85a057"
keywords: [DaXBench, DaX, JAX, differentiable simulation, deformable object manipulation, MLS-MPM, mass-spring, OpenAI Gym, APG, SHAC, PPO, ILD, Transporter, CEM-MPC, WhipRope, PourSoup, Fold-Cloth, Push-Rope, Kinova Gen3, sim-to-real]
domain: "Robotics"
code_url: "https://github.com/AdaCompNUS/DaXBench"
cited_by: []
---

## Problem

Deformable Object Manipulation (DOM) — rope, cloth, fluid, elastoplastic — is bottlenecked by simulator quality. Existing DOM benchmarks face two problems: (i) most are **single-object** (one engine, one material class), so cross-task generalization of an algorithm is untestable; and (ii) most are **non-differentiable** (e.g. SoftGym), so they cannot evaluate the recent class of differentiable-physics-based methods (SHAC, APG, ILD, Diff-MPC) that exploit analytic gradients through the simulator. This leaves two open questions: (1) does a task-specific DOM algorithm transfer to other tasks, and (2) does a differentiable-physics-based algorithm beat its non-differentiable counterpart in general?

## Key idea

Build **DaXBench**: a single differentiable DOM benchmark with **9 tasks** spanning rope, cloth, fluid, and elastoplastic objects, all wrapped in a JAX-based simulator (DaX) that exposes batched parallel rollouts and end-to-end gradients via OpenAI Gym. Then run a controlled bake-off across **8 algorithms** drawn from three paradigms (planning, imitation learning, reinforcement learning) — including their gradient-free vs. gradient-based variants — to map where differentiable physics actually helps.

## Method

- **Simulator (DaX)**: implemented in [JAX](https://github.com/google/jax). Rope and liquid use Moving Least Squares Material Point Method (MLS-MPM, particle-based), cloth uses a [[mass-spring-system]] (mesh-based). Two memory tricks: **lazy dynamic update** (only update the active region around the manipulator — ~2 orders of magnitude speed-up; e.g. rope active grid 32x6x32 vs. full 128x128x128) and **gradient checkpointing** (re-compute forward subsegments during backward). Action-not-in-contact gradients are repaired by **local action adjustment** so the gripper always touches the object — eliminates discontinuous gradients.
- **Tasks (9)**: liquid (Pour-Water, Pour-Soup), rope (Push-Rope, Whip-Rope), cloth (Fold-Cloth-1, Fold-Cloth-3, Fold-T-shirt, Unfold-Cloth-1, Unfold-Cloth-3). Tasks are split into **high-level macro-action** (pick-and-place, push) and **low-level control** (raw Cartesian gripper velocities). Whip-Rope and Pour-Water/Pour-Soup are intentionally low-level: macro-actions cannot express the high-frequency momentum control they require.
- **Algorithms benchmarked (8)**:
  - Planning — CEM-MPC (gradient-free), Diff-MPC (random-init differentiable MPC), Diff-CEM-MPC (CEM init + differentiable refinement)
  - Imitation Learning — Transporter (RGBD/particles), ILD (differentiable physics)
  - Reinforcement Learning — PPO (gradient-free), SHAC (Short-Horizon Actor-Critic, differentiable), APG (Analytic Policy Gradients, differentiable)
- **Reward**: $r = r_{\text{gt}} + r_{\text{aux}}$ during training, with $r_{\text{gt}} = \exp(-\lambda D(s', g))$ matching object-particle positions to the goal $g$ and $r_{\text{aux}} = \exp(-L_2(s,a))$ encouraging the gripper to contact the object. Evaluation uses $r_{\text{gt}}$ only.
- **Sim-to-real probe**: deploy CEM-MPC with DaX as the predictive model on Push-Rope on a Kinova Gen3 robot.

## Results

**Throughput.** 128 batched rollouts x 80 timesteps (forward + backward) in ~3 s on 4 x 2080-Ti GPUs.

**RL bake-off (final reward, mean ± std-err over 20 seeds).**

| Task type | Task | PPO | APG | SHAC |
|---|---|---|---|---|
| High-level | Fold-Cloth-1 | **0.40 ± 0.13** | 0.36 ± 0.06 | 0.34 ± 0.07 |
| High-level | Fold-Cloth-3 | 0.21 ± 0.11 | 0.19 ± 0.09 | **0.22 ± 0.22** |
| High-level | Fold-T-shirt | **0.61 ± 0.08** | 0.40 ± 0.05 | 0.44 ± 0.09 |
| High-level | Unfold-Cloth-1 | **0.72 ± 0.01** | 0.42 ± 0.02 | 0.50 ± 0.03 |
| High-level | Unfold-Cloth-3 | **0.56 ± 0.00** | 0.39 ± 0.02 | 0.48 ± 0.03 |
| High-level | Push-Rope | **0.75 ± 0.02** | 0.72 ± 0.02 | **0.75 ± 0.01** |
| **Low-level** | **Whip-Rope** | **0.25 ± 0.10** | **0.83 ± 0.01** | **0.66 ± 0.03** |
| Low-level | Pour-Water | 0.27 ± 0.02 | 0.27 ± 0.02 | 0.28 ± 0.00 |
| Low-level | Pour-Soup | 0.27 ± 0.08 | 0.27 ± 0.00 | 0.32 ± 0.13 |

**Headline:** APG dominates Whip-Rope (0.83) by **3.3x** over PPO (0.25), and SHAC (0.66) is also above PPO. The opposite ordering holds on most high-level macro-action tasks: PPO is best or tied. The interpretation is that gradient-based RL only beats sample-based RL when the optimization landscape is **smooth and convex** (Whip-Rope's optimal control is a smooth motion trajectory) and when the policy needs **fine-grained low-level control**; on macro-action tasks the landscape is non-smooth and the gradient-based methods get stuck without entropy-regularized exploration.

**Imitation Learning (20 rollouts).** ILD (differentiable physics + step-wise expert gradients) substantially beats Transporter on every macro-action task it can be evaluated on (Transporter lacks low-level support):
- Fold-Cloth-3: 0.82 vs 0.40, Fold-T-shirt: 0.59 vs 0.46, Unfold-Cloth-1: 0.64 vs 0.48, Unfold-Cloth-3: 0.52 vs 0.30. ILD also struggles with **unbalanced state representations** (1000x3 particles vs 6 gripper states) on Pour-Water (0.32) and Pour-Soup (0.42) — the particle signal overwhelms the gripper signal.

**Planning.** Diff-CEM-MPC ≥ CEM-MPC ≥ Diff-MPC across nearly all tasks. Random-init Diff-MPC (e.g. 0.14 on Fold-Cloth-3, 0.30 on Pour-Water) collapses because random control sequences are far from any local optimum on a non-smooth/non-convex landscape; CEM-init solves this by giving the differentiable refinement a near-optimal starting point.

**Sim-to-real.** CEM-MPC + DaX deployed on a Kinova Gen3 produces real-robot Push-Rope trajectories that closely track DaX-predicted next states across 6 push steps (qualitative; supplementary video).

## Limitations

- **No closed-loop visuomotor policies** — observations are particle-level state, not RGBD; perception transfer is not stress-tested. (Transporter with RGB is reported in appendix only.)
- **No bending-twisting fidelity for rope** — MPM rope is a chain of particles without explicit twist DOFs; cannot represent torsion-dominated dynamics (knots, suturing). Cosserat-style alternatives sit outside the benchmark.
- **Cloth uses [[mass-spring-system]]**, which is fast and differentiable but cannot reproduce bending stiffness or anisotropic materials accurately.
- **Sim-to-real evaluation is qualitative**: only Push-Rope on a single robot, no quantitative tip-to-goal metric, no baseline simulator comparison.
- **All differentiable RL baselines are not entropy-regularized**, so the "gradient-based RL is bad on macro-action tasks" finding may partly reflect missing exploration regularization rather than a fundamental gradient pathology.
- **Local action adjustment** (always-in-contact) is a strong inductive bias that may not be realistic for real-robot deployment (e.g. throwing, free flight).

## Open questions

- How much of APG's Whip-Rope win is due to MLS-MPM particle physics vs. due to JAX-batched gradient flow? Would the same algorithm dominate on a Cosserat-rod or linked-capsule rope?
- Can entropy-regularized differentiable RL (e.g. add policy entropy bonus to APG) close the gap on high-level macro-action tasks?
- Does the lack of bending-twisting torsion in MPM rope affect downstream tasks like knot-tying, where Cosserat-style rod models matter? (Sibling work [[deform-differentiable-discrete-elastic-rods-real]] makes the case that DER is needed.)
- Differentiability is shown to enable system identification (compare sim/real observations, gradient through simulator parameters); how robust is this in practice across material classes?

## My take

DaXBench is the most influential broad-coverage differentiable DOM benchmark in the JAX ecosystem and the natural complement to PyTorch-side differentiable rope simulators like [[deform-differentiable-discrete-elastic-rods-real]]. For DLO-specific work, the key contribution is **WhipRope as a clean, low-level, smooth-landscape rope task where APG (analytic policy gradients) demonstrably dominates** — this is the cleanest published example of differentiable physics paying off for dynamic rope control under matched algorithm budgets. The most actionable downstream signal is: when the optimization landscape is well-behaved and the action space is low-level continuous control, prefer APG/SHAC over PPO; when macro-actions are available and the landscape is non-smooth, PPO remains the safer default. The simulator's MPM rope is a real limitation for torsion-heavy tasks, but for dynamic swinging/whipping it is sufficient. For DeformY: the WhipRope baseline is the most direct rival to consider for our closed-loop full-3D tip-targeting comparison, and the right way to extend it is to swap MLS-MPM rope for a Cosserat formulation while keeping the JAX-batched APG training loop.

## Related

- [[dynamic-deformable-object-simulation]]
- [[model-based-planning-for-manipulation]]
- [[gradient-inaccessibility-contact-mediated-manipulation]] — the counterweight to WhipRope's APG result: [[dlo-lab-benchmarking-deformable-linear-object]] (ICML 2026) finds analytic gradients *losing* to CMA-ES on 7 of 8 DLO tasks wherever the reward depends on an object not yet contacted.
**Foundations used**
- [[deformable-linear-object]] — rope is one of the four object classes
- [[mass-spring-system]] — used to model cloth dynamics in DaX
- [[finite-element-method]] — comparison baseline class for non-MPM cloth modeling
- [[sim-to-real-transfer]] — the benchmark's empirical reach claim
- [[model-based-reinforcement-learning]] — CEM-MPC and Diff-CEM-MPC are model-based planners
- [[imitation-learning]] — ILD and Transporter
- [[behavioral-cloning]] — backbone of Transporter

**Concepts introduced**
- [[differentiable-deformable-benchmark]] — the JAX-based MPM + mass-spring DOM benchmark pattern with batched gradient flow

**Claims supported**
- [[apg-dominates-shac-and-ppo-on-whiprope-low-level]]
- [[jax-differentiable-rope-enables-batched-rl-vs-cem-mpc]]

**Important referenced work** (not yet ingested — candidates for follow-up `/ingest`)
- SoftGym (Lin et al.) — non-differentiable DOM benchmark; immediate precursor.
- PlasticineLab (Huang et al.) — single-object differentiable benchmark for elastoplastic.
- DiSECt (Heiden et al.) — differentiable cutting; single-task.
- ChainQueen (Hu et al.) — early differentiable MPM simulator.
- DiffTaichi (Hu et al.) — differentiable physics framework underlying many simulators.
- SHAC (Xu et al.) — Short-Horizon Actor-Critic; one of the differentiable RL baselines.
- APG / Brax (Freeman et al.) — Analytic Policy Gradients; the winning low-level RL baseline on Whip-Rope.
- ILD (Chen et al.) — differentiable-physics imitation learning baseline.
- Transporter (Zeng et al.) — pick-and-place imitation learning baseline.

**Sibling DLO works** (this wiki)
- [[deformx-versatile-co-simulation-framework-deformable]] — the Cosserat-rod + Isaac Sim counterpart; complementary architecture for DLO simulation, with explicit Cosserat physics where DaXBench uses MLS-MPM particles.
