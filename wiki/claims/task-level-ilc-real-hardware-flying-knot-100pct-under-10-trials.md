---
title: Task-Level ILC from one human demo plus a 5-parameter rope model achieves 100% success in fewer than 10 real-hardware trials of the flying knot across 7 rope and cable types
slug: task-level-ilc-real-hardware-flying-knot-100pct-under-10-trials
status: supported
confidence: 0.75
tags:
- ILC
- iterative-learning-control
- deformable-manipulation
- rope
- real-world-learning
- sample-efficiency
- xArm-7
- flying-knot
domain: Robotics
source_papers:
- '[[learning-deformable-object-manipulation-using-task]]'
evidence:
- source: learning-deformable-object-manipulation-using-task
  type: supports
  strength: strong
  detail: Suresh & Atkeson 2026 evaluate Task-Level ILC with a critical-point objective on xArm 7 across 7 rope types (chain, latex tubing, braided, twisted), 7-25mm thickness, 0.013-0.5 kg/m density. Every rope reaches a successful flying knot within 10 trials. A 40-trial robustness test on the converged command per rope shows 0 failures (100% success).
conditions: Tying a one-handed Overhand-style flying knot on a 1.1m rope with a fixed end weight, on xArm 7 with 250Hz command and 200Hz Vicon rope tracking. Single human demonstration with manually-annotated rope-self-collision frame as the critical point. Approximate 11-link point-mass-with-bending rope model (5 free parameters). Drake/Clarabel QP inverse model with linearized joint position, velocity, acceleration, and torque constraints. The 100% figure is for repeated execution of the converged command, not for re-running learning end-to-end.
date_proposed: 2026-05-06
date_updated: 2026-05-06
---
## Statement

Given (i) a single human demonstration of a flying knot, (ii) a rope model that has only 5 hand-tuned parameters and is qualitatively wrong about post-collision behavior, and (iii) an optimization-based inverse model with a critical-point objective, Task-Level Iterative Learning Control on real xArm 7 hardware converges to a successful flying-knot command within fewer than 10 real trials for every one of the 7 rope and cable types tested, and the converged command then succeeds on 40/40 repeated trials.

This combines two separate empirical findings from the same paper into one claim: (1) every rope succeeds within the 10-trial budget, and (2) the post-convergence command is repeatable.

## Evidence summary

- **Direct support: 7-rope evaluation** ([[learning-deformable-object-manipulation-using-task]], Section "Learning Across Rope Types", Figure all_rope_sequence). 7 rope types (chain, latex surgical tubing, braided, twisted), spanning 7-25mm thickness and 0.013-0.5 kg/m density. Every rope hits a successful flying knot within 10 trials; some succeed in 4 trials, none exceed 10. The same 5-parameter rope model is used across all 7 ropes — only the end-mass is tuned per rope to enable the human demonstrator. The paper uses a single demonstration on rope 1 for all results.
- **Direct support: post-convergence repeatability** (Section "Learned Command Success Rate"). After the first successful trial, the converged command is run 40 additional times and produces 0 failures. This grounds the "100% success" framing.
- **Indirect support: ablation against equal-weighting objective** (Section "Weighted Error Objectives", Figure critvseq). With equal weighting and the same model and demo, learning fails — the critical-point objective is not optional, it is *necessary*. This is what gives the claim its specific shape: it is "Task-Level ILC *with critical-point objective*" that works, not Task-Level ILC in general.
- **Indirect support: model-parameter robustness** (Table II, Section "Sensitivity to Model Parameters"). Order-of-magnitude variation in stiffness ($k$) and three orders of magnitude in end mass ($m_e$) leave learning success intact for most settings, including a sweep from $10^3$ to $10^5$ in stiffness.

## Conditions and scope

- *Task*: one-handed Overhand-style flying knot on a 1.1m rope.
- *Hardware*: xArm 7 (7-DoF arm), 250Hz command, 100:1 gear ratio so rope-on-arm reaction torque is negligible.
- *Sensing*: Vicon at 200Hz with 11 retroreflective markers per rope plus 4 hand markers; rope state at the critical point must be measurable before self-occlusion.
- *Demonstration*: a single human demonstration with a manually-annotated critical point ($t_c =$ rope-self-collision instant).
- *Rope model*: 11-link point-mass chain with bending stiffness, damping, and end mass; 5 free parameters; maximal-coordinate variational integrator from Lavalle-Tedrake.
- *Inverse model*: Drake-formulated QP, Clarabel solver; linearized joint position/velocity/acceleration/torque constraints.
- *"100%" qualifier*: means 40/40 trials of the converged command, not a re-run of the learning loop. The first 1-9 trials of learning are by definition *unsuccessful*; the claim is about the post-convergence regime.

## Counter-evidence

- *None published as of 2026-05-06.* The claim is a single-paper, single-lab result; no independent replication. Confidence is held below 0.85 for that reason.
- *Robot/sensor specificity*: the work has only been demonstrated on xArm 7 with Vicon; transfer to less-precise sensing or other arm morphologies is unestablished.
- *Failure-of-learning footnote*: the paper notes that for some transfer pairs (rope 5/6 → 2/3) the algorithm does not converge in 10 trials. This is a *transfer* failure, not a *learning-from-scratch* failure, and does not contradict this claim — but it shows that the 10-trial bound is not unconditional.

## Linked ideas

(Linked ideas may be appended later by `/ideate`.)

## Open questions

- Does the bound hold on a less-precise sensor stack (e.g. RGB-D rope tracking instead of Vicon)?
- Does the bound hold for different demonstrators (the paper's "demonstration variation" experiment varies demos but is on a single human)?
- Does the bound hold for harder knots (figure-eight, bowline) on the same hardware?
- What is the dependence on rope length? All experiments use 1.1m.
