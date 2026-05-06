---
title: "DaXBench Whip-Rope baselines: APG 0.83 dominates SHAC 0.66 and PPO 0.25 — analytic policy gradients win on dynamic deformable tasks"
slug: "apg-dominates-shac-and-ppo-on-whiprope-low-level"
status: supported
confidence: 0.7
tags: [DLO, DOM, deformable-object-manipulation, RL, differentiable-physics, APG, SHAC, PPO, benchmark, low-level-control]
domain: "Robotics"
source_papers: ["[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]"]
evidence:
  - source: "[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]"
    type: supports
    strength: strong
    detail: "DaXBench Whip-Rope (low-level Cartesian-velocity control of a gripper holding one end of a 70-step rope) RL baselines, mean ± std-err over 20 random seeds: APG 0.83 ± 0.01, SHAC 0.66 ± 0.03, PPO 0.25 ± 0.10. APG (analytic policy gradients) achieves a 3.3× score advantage over PPO and 1.26× over SHAC on the only low-level rope task in the benchmark. The same algorithms, run with the same training budget, do not show this gap on high-level macro-action tasks (PPO 0.40/0.21/0.61/0.72/0.56/0.75 on the six macro-action tasks frequently matches or beats APG/SHAC). The paper's interpretation is that the Whip-Rope optimal control is a smooth motion trajectory on a well-behaved gradient landscape, where analytic gradients shine; macro-action tasks have non-smooth/non-convex landscapes where unregularized gradient-based RL collapses to local optima. Reward function: r_gt = exp(-lambda * D(s', g)) on rope particle positions + r_aux for gripper contact, evaluation with r_gt only."
conditions: "DaXBench DaX simulator (JAX + MLS-MPM rope, batched parallel rollouts on 4 x 2080-Ti GPUs); rope = particle-based MPM (no explicit twist DOF); single rope length and material; Whip-Rope task with 3D Cartesian-velocity action space and 70-step horizon; RL training budget held constant across PPO, SHAC, APG; baselines without entropy regularization; evaluation over 100 RL rollouts per setting (paper Reproducibility Statement) with 20 seeds; particle-state observation space (no images)."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

On the DaXBench Whip-Rope task — low-level Cartesian-velocity control of a gripper whipping a rope to a target configuration over 70 steps — Analytic Policy Gradients (APG) achieves substantially higher final reward than Short-Horizon Actor-Critic (SHAC) and Proximal Policy Optimization (PPO) under matched training budgets. Concretely: APG 0.83, SHAC 0.66, PPO 0.25 (final reward, mean ± std-err over 20 seeds). The win is **task-conditional**: the same APG/SHAC/PPO trio does not show this ordering on high-level macro-action tasks in the same benchmark; on most of those PPO matches or exceeds the differentiable baselines.

## Evidence summary

DaXBench evaluates three RL methods (PPO, SHAC, APG) on nine DOM tasks with a uniform reward function and reproducibility protocol. Whip-Rope is the only **low-level continuous-control rope task** in the benchmark and the cleanest case where the optimal policy is a smooth trajectory in action space — exactly the setting where differentiable physics provides reliable analytic gradients. APG outperforms SHAC and PPO by margins (0.83 vs 0.66 vs 0.25) that are (i) much larger than the per-method standard error (0.01, 0.03, 0.10) and (ii) reversed or absent on the six high-level macro-action tasks in the same benchmark. This task-dependence is itself part of the claim: differentiable RL is not unconditionally better than PPO; it dominates specifically on smooth-landscape low-level control.

A complementary signal from the planning baselines: Diff-CEM-MPC (CEM-init + differentiable refinement) consistently beats CEM-MPC and dramatically beats Diff-MPC (random-init differentiable MPC, scoring only 0.37 on Whip-Rope vs 0.34 for CEM-MPC and 0.33 for Diff-CEM-MPC). The pattern is consistent — differentiable physics adds real signal on top of a near-optimal initialization but cannot rescue a random-init policy on a non-smooth landscape.

## Conditions and scope

The claim applies under the conditions in `conditions` above. Specifically:

- The simulator's rope physics is **MLS-MPM particles** (no explicit bending or twist DOF). The result may differ on a Cosserat-rod simulator (sibling work [[deform-differentiable-discrete-elastic-rods-real]] argues Cosserat/DER changes the gradient quality).
- The action space is **3D Cartesian velocity**; macro-action variants are not tested for Whip-Rope (they are infeasible by task design).
- All three RL baselines run **without entropy regularization**. An entropy-regularized variant of APG/SHAC could change relative ordering on tasks where exploration matters; it is unlikely to flip Whip-Rope where exploration is not the bottleneck.
- Evaluation is **simulator-internal** — there is no real-robot Whip-Rope deployment, so this is not yet a sim-to-real claim. (DaXBench's only real-robot deployment is Push-Rope with CEM-MPC.)
- The claim does not extend to other low-level continuous-control DOM tasks: on Pour-Water and Pour-Soup (also low-level), APG/SHAC/PPO are all near 0.27, so the APG advantage does not generalize to all low-level tasks — only to those with a smooth, well-behaved optimization landscape.

## Counter-evidence

None directly observed in the source paper. Plausible counter-stories:

1. **Reward shaping is responsible.** The auxiliary contact reward $r_{\text{aux}} = \exp(-L_2(s, a))$ may give APG a stronger signal when the gripper's smooth swing trajectory naturally maintains contact, while PPO without explicit contact-shaping wastes samples. A version with $r_{\text{gt}}$-only training could shrink the gap.
2. **PPO under-tuned.** PPO benefits enormously from hyperparameter tuning; the published 0.25 may not be PPO's ceiling.
3. **APG lucky on smooth landscape.** Whip-Rope's optimal control happens to be smooth; on a slightly perturbed task variant (e.g. obstacle, contact-rich), APG could collapse.
4. **JAX implementation quality.** All three baselines are JAX-implemented and share infrastructure; if the JAX-PPO implementation is less polished than the differentiable-RL implementations, the gap may narrow with a stronger PPO baseline.

## Linked ideas

(none yet — likely follow-ups: comparing APG on Cosserat-rope vs MPM-rope; entropy-regularized differentiable RL as a single algorithm covering both macro and low-level tasks.)

## Open questions

- Does the APG advantage hold on a Cosserat-rod rope simulator with explicit bending/twist?
- How does the gap scale with horizon? (Whip-Rope is 70 steps; longer horizons may compound APG's gradient pathologies.)
- Does APG still dominate when the policy must close the loop on partial observations (image-only, no particle state)?
- What entropy / exploration regularizer recovers PPO-level performance on macro-action tasks while preserving APG's win on low-level smooth tasks — i.e. one algorithm that wins everywhere?
