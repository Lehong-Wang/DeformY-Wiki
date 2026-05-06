---
title: "DIDP reaches 84.3% within 5 cm and 20.8% within 1 cm tip-to-goal on a 3D reduced-order Cosserat rope-whipping benchmark"
slug: "didp-3d-rope-tip-targeting-success-rates"
status: supported
confidence: 0.7
tags: [DLO, diffusion-policy, benchmark, success-rate, sim-only, 3D-rope-manipulation, GVS]
domain: "Robotics"
source_papers: ["[[dynamic-manipulation-deformable-objects-3d-simulation]]"]
evidence:
  - source: "[[dynamic-manipulation-deformable-objects-3d-simulation]]"
    type: supports
    strength: moderate
    detail: "DIDP (Kinematics+Dynamics + DDIM + TTA, project-only finetune) reports mean tip-to-goal distance 3.6 cm with success rates 93.9% @10cm, 84.3% @5cm, 62.3% @2cm, 20.8% @1cm on a 55,000-trajectory 3D rope-whipping benchmark built on a reduced-order GVS Cosserat-rod simulator (T=0.5s, 500 timesteps, 21 material points, 2-joint actuation). Single-run, no error bars. No real-robot evaluation."
conditions: "Simulation only, single rope material/geometry. 2-joint continuum-soft-robot actuation with action $\\boldsymbol{q} \\in \\mathbb{R}^{2\\times4}$ piecewise-constant over 4 segments; 55k expert trajectories from RK4 / ODE15s GVS forward simulation. Test-time adaptation restricted to the final projection layer. Goal is a Cartesian tip position; success defined by Euclidean distance threshold."
date_proposed: 2026-05-06
date_updated: 2026-05-06
---

## Statement

On the **3D rope-whipping benchmark introduced in DIDP** — 55,000 trajectories of a 2-joint continuum soft robot simulated with a 20-DoF reduced-order GVS Cosserat-rod model — the **DIDP framework** (full-system diffusion policy with IL+TO pretraining and physics-informed test-time adaptation) reaches:

| Threshold | Success rate |
|---|---|
| within 10 cm | 93.9% |
| **within 5 cm** | **84.3%** |
| within 2 cm | 62.3% |
| within 1 cm | 20.8% |

with mean tip-to-goal distance **3.6 cm**.

These are the first headline numbers for **3D goal-conditioned dynamic manipulation** of deformable linear objects under a unified policy framework. The result is **simulation-only**.

## Evidence summary

The DIDP paper (arXiv 2505.17434) reports the above numbers in its main quantitative table (Table on Learning Space) under the configuration Kinematics+Dynamics + DDIM + TTA. Three companion ablations support the result's internal coherence:

- Without TTA, DIDP still produces 80.0% @5cm and 19.0% @1cm — the bulk of the gain comes from extending the policy's state to the full 20-DoF reduced-order system, not from test-time adaptation.
- IL alone (no trajectory-optimization finetune) → 80.0% @5cm; TO alone (from scratch) collapses to 6.8% @5cm; **IL+TO** is what drives the headline 84.3%.
- DDP (physics goal-loss) without KBC (start-state regularization) **degrades** performance to 12.4% @5cm; both PITA losses are needed.

The benchmark is well-specified: deterministic RK4 forward simulation, ODE15s stiff solver for divergent-trajectory filtering, 21-point Cartesian state, 4-segment piecewise actuation drawn from $\boldsymbol{q}_1\sim\mathcal{U}[-\pi,\pi]$, $\boldsymbol{q}_2\sim\mathcal{U}[-\pi/2,\pi/4]$. Goal Euclidean distance is the only metric; success thresholds are reported at four levels.

## Conditions and scope

The 84.3% / 20.8% figures hold under the following:

- **Simulation only** on the GVS reduced-order rod simulator. No physical robot.
- **Single rope material and geometry.** The paper trains and evaluates on one rope; cross-rope generalization is not measured.
- **Open-loop action policy with fixed horizon** ($N=500$). No closed-loop visual feedback, no replanning under disturbance.
- **Cartesian tip-position goal.** No orientation, multi-target, or topological goals.
- **DDIM sampling** with $T=512$ training steps, project-only TTA, ~10s/sample inference.
- **Single run, no error bars.** Authors explicitly answer "No" on the NeurIPS Statistical Significance question.

## Counter-evidence

None directly observed in the source paper. The plausible counter-stories are:

1. **Sim-only performance does not transfer.** The DeformX experience (sibling paper) shows that calibrated rod physics on a different DLO task still leaves a 3-5x sim-to-real gap for linked-capsule baselines, and even the Cosserat-Isaac backend has measurable real-world residuals. DIDP's headline number on a hardware whipping rig may be substantially worse — and is currently unmeasured.
2. **Single rope, single seed.** Variance across seeds is unreported; the @1cm rate of 20.8% is close to the @1cm rate of an IL-only baseline (19.0%), so the precision gain from PITA at the tightest threshold may not be statistically significant.
3. **Benchmark is self-defined.** The simulator and the policy share an authoring team. Independent reproduction on the same benchmark, or evaluation on an external benchmark, is not yet available.

## Linked ideas

(none yet — DeformY's planned closed-loop sim-to-real DLO benchmark is the natural follow-on extension, but it is not yet a wiki idea page.)

## Open questions

- What is the sim-to-real gap when a DIDP policy trained on this GVS benchmark is deployed on a physical 2-joint robot with a real rope?
- How do the success rates degrade across rope materials (hemp / nylon / rubber) and lengths? The paper drafts a 3-class material schema but disables it in the released configuration.
- Can the @1cm rate (20.8%) be lifted without inflating inference cost? The headline 84.3%@5cm is competitive but the sub-cm regime is still poor.
- Does adding closed-loop visual feedback improve robustness to action perturbations during execution, or does it disrupt the IL prior more than it helps?
