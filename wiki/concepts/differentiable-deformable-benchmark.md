---
title: "Differentiable Deformable-Object Benchmark"
aliases: ["differentiable DOM benchmark", "JAX deformable benchmark", "batched differentiable DOM simulator", "differentiable physics benchmark for DOM", "DaXBench-style benchmark"]
tags: [DLO, DOM, deformable-object-manipulation, differentiable-physics, JAX, MPM, mass-spring, benchmark, RL, imitation-learning, planning]
maturity: emerging
key_papers: ["[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]", "[[dlo-lab-benchmarking-deformable-linear-object]]"]
first_introduced: "2023"
date_updated: 2026-07-30
related_concepts: ["[[gradient-inaccessibility-contact-mediated-manipulation]]"]
parent_topic: "[[dynamic-deformable-object-simulation]]"
---

## Definition

A **differentiable deformable-object benchmark** is a multi-task DOM simulation suite with three jointly-required properties:

1. **End-to-end differentiability** through deformable-object dynamics (analytic gradients of state w.r.t. control or simulator parameters available at GPU-kernel level), so analytic policy gradient methods (APG, SHAC), differentiable model-predictive control (Diff-MPC), and differentiable imitation learning (ILD) can be evaluated without bolting on a learned surrogate.
2. **Multi-object coverage** — at minimum rope + cloth + (liquid or elastoplastic) — under one consistent task API, so the question "does method X transfer across object classes" becomes testable by holding the simulator fixed.
3. **Batched parallel rollouts** at machine-learning-grade throughput (hundreds of environments in parallel, sub-second per step on commodity GPUs), so on-policy RL has enough wall-clock budget to converge and gradient-based methods have enough samples for variance reduction.

The pattern is exemplified by DaXBench (JAX + MLS-MPM for liquid/rope + [[mass-spring-system]] for cloth), and contrasts with the dominant prior pattern of single-object simulators (PlasticineLab, DiSECt, DiffSim) and non-differentiable multi-object benchmarks (SoftGym, ThreeDWorld).

## Intuition

Differentiable physics adds analytic gradients of dynamics to the policy training loop. To benefit from them you need:

- enough *task variety* that algorithm comparisons are not over-fit to one object;
- enough *throughput* that batched rollouts are feasible;
- *consistent reward functions* (typically particle-position match $D(s', g)$ + auxiliary contact reward) so the gradient signal is comparable across tasks.

Single-object differentiable benchmarks satisfy property (1) but not (2); non-differentiable multi-object benchmarks satisfy (2) but not (1). Putting all three together is the contribution.

## Formal notation

Per task: state $s \in \mathbb{R}^{N \times d}$ (particles or mesh vertices), action $a \in \mathbb{R}^{a}$ (gripper velocity / pick-and-place), forward dynamics $s_{t+1} = f(s_t, a_t)$ with $\partial f / \partial a_t$ and $\partial f / \partial s_t$ available analytically. Reward
$$ r(s, a) = \exp(-\lambda D(s', g)) + \exp(-L_2(s, a))$$
where the second term encourages contact and prevents zero-gradient regions. Batched rollout: $B$ environments stepped jointly via $\texttt{vmap}$ (in JAX) or equivalent.

## Variants

- **JAX-MPM + mass-spring** (DaXBench): particle MLS-MPM for rope/liquid, mass-spring for cloth. Rich object coverage, no torsion DOF.
- **Differentiable Cosserat rope** (sibling: [[deform-differentiable-discrete-elastic-rods-real]]): drops particle MPM rope in favor of Cosserat / DER for explicit bending-twisting fidelity. Narrower coverage (rope only) but better physics for DLO-specific work.
- **Differentiable cloth FEM** (DiffCloth, DiffSim): mesh + finite-element bending; richer cloth physics, no rope or liquid.
- **Differentiable elastoplastic** (PlasticineLab, ChainQueen): material point method for plastic; single-object.
- **Differentiable cutting** (DiSECt): differentiable rupture; specialized.

## Comparison

- vs. **non-differentiable benchmarks** (SoftGym, ThreeDWorld): non-differentiable benchmarks cannot evaluate APG/SHAC/ILD/Diff-MPC at all; they are useful only for gradient-free planning, IL, and standard RL.
- vs. **single-object differentiable simulators**: single-object simulators answer "does method X work on this task" but not "does it generalize across DOM" — exactly the gap multi-task differentiable benchmarks close.
- vs. **rigid-body differentiable benchmarks** (Brax, MuJoCo MJX): rigid-body benchmarks have mature differentiable engines, but DOM dynamics (MPM, mass-spring, FEM) introduce contact discontinuities and high-DoF state spaces that rigid benchmarks do not exercise.

## When to use

- The research question is "does differentiable physics help DOM?" and you need a controlled comparison across multiple object classes.
- You are designing a new differentiable DOM algorithm and need a standardized place to report against APG, SHAC, PPO, ILD, Transporter, CEM-MPC.
- You want batched on-policy RL training for DOM at scales where single-environment simulators are infeasible.

Skip this pattern when:

- The DOM problem is bending-twisting-dominated (knot tying, suturing) — particle MPM rope misses torsion DOFs and a Cosserat formulation is closer to first principles.
- Sim-to-real fidelity matters more than algorithm comparison (calibration to a real rope on a real robot is rarely the benchmark's strength).

## Known limitations

- Particle MPM rope misses explicit twist; cloth mass-spring misses bending stiffness — physics realism trades against differentiability and speed.
- All baselines must accept the benchmark's reward shaping (particle-position match + contact aux) and observation interface (full particle state); methods with strong perception-side claims (image-based, sparse-view) are bottlenecked.
- Benchmark scores are highly sensitive to local action adjustment, gradient checkpointing intervals, and reward weighting — small implementation choices change the rank order.
- Most reported sim-to-real evaluation is qualitative (single robot, no simulator-comparison ablation).

## Open problems

- Adding entropy regularization or explicit exploration to differentiable RL baselines to fix their failure mode on high-level macro-action tasks.
- Replacing particle MPM rope with a differentiable Cosserat / DER backbone while preserving JAX-batched rollouts.
- Quantitative cross-simulator sim-to-real benchmark on the same physical setup (e.g. one rope, multiple simulators, same policy).
- Standardized image-based observation channel so vision-conditioned policies can be benchmarked head-to-head.

## Key papers

- [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] — defines the pattern; 9 tasks across rope/cloth/liquid/elastoplastic; 8 algorithms; JAX + MLS-MPM + mass-spring; APG dominates Whip-Rope, PPO dominates macro-action tasks; sim-to-real on Push-Rope.

## My understanding

For DLO-focused research, this concept is mostly useful as a **rival to compare against** rather than a substrate to extend, because the particle MPM rope is the wrong physical model for torsion-heavy DLO tasks. The lasting contribution is the **task design and algorithmic protocol**: nine tasks split by macro-action vs. low-level control, a controlled bake-off of gradient-based and gradient-free methods within each paradigm, and clean reporting of where differentiable physics actually pays off (low-level smooth-landscape control like Whip-Rope) vs. where it does not (high-level macro-action). Future DLO benchmarks should preserve that protocol while swapping the rope physics to Cosserat — that is what DeformY's evaluation should look like.
