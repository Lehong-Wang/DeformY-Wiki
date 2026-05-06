---
title: "Learning Deformable Object Manipulation Using Task-Level Iterative Learning Control"
slug: "learning-deformable-object-manipulation-using-task"
arxiv: "2602.21302"
venue: "arXiv.org"
year: 2026
tags: [DLO, rope, dynamic-manipulation, iterative-learning-control, ILC, real-world-learning, flying-knot, knot-tying, xArm-7, model-based, single-demo, transfer, robot-learning]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "23442b60c64a15dfb0c16edfbb605f429d9f23f1"
keywords: [Task-Level ILC, critical point objective, flying knot, xArm 7, deformable object manipulation, model-based ILC, optimization-based inverse model, QP, Drake, Clarabel, Bezier command, point-mass rope model, transfer across ropes]
domain: "Robotics"
code_url: "https://flying-knots.github.io"
cited_by: []
---

## Problem

Dynamic manipulation of a rope is hard for the same reason it is hard for humans: the rope has effectively infinite degrees of freedom, is severely underactuated (one hand drives one end of an N-link chain), exhibits unmodeled contact (rope-on-rope, rope-on-finger), and has no closed-form accurate dynamics for the regime of interest. Behavior cloning needs many demonstrations and does not transfer across ropes; sim-to-real with domain randomization pays a worst-case robustness tax that erodes performance whenever the *actual* rope dynamics differ from any single point in the randomization range. Classical model-based Iterative Learning Control (ILC) corrects feedforward commands trial-by-trial using an approximate inverse model, but on a rope, **equally-weighted task tracking fails** — driving down errors at non-decisive parts of the trajectory wastes update budget and increases error at the moment that actually matters. The paper asks: can a robot learn a hard dynamic deformable-object task on real hardware in single-digit trials, from one demonstration plus a hand-crafted but loose rope model, in a way that transfers across rope types?

## Key idea

Adapt **Task-Level ILC** to deformable-object manipulation by combining two ideas:

1. **Critical-point objective.** Instead of weighting state-tracking error equally along the trajectory, weight it only at a manually-annotated *critical point* (here: the moment of rope self-contact in the demonstration). Errors before and after that instant are dropped from the cost. This focuses the inverse-model update on the few decisive degrees of freedom and incidentally avoids modeling post-contact rope physics altogether.
2. **Task-level (not robot-level) state.** The task error is defined on the manipulated *object* (rope point-mass positions), not on the robot's joint trajectory. The inverse model maps rope-state error at $t_c$ back to a feedforward 7-DoF arm-joint Bezier command via a Quadratic Program over the linearized system dynamics.

Together they let a single human flying-knot demo plus a deliberately *loose* point-mass-with-bending-stiffness rope model bootstrap a learning loop that converges in fewer than 10 real trials per rope.

## Method

- **Task: flying knot.** A one-handed dynamic motion that creates a loop and flips the rope's end through it to tie an Overhand Knot. The *critical point* $t_c$ is the moment of rope self-collision in the demonstration; rope state at $t_c$ is the learning target.
- **Command parameterization.** $\mathbf{u}(t) \in \mathbb{R}^{10 \times 8}$ — 10 Bezier curves (7 xArm 7 joints + 3 base translations, base translations clamped to zero), 8 knot points equally spaced in time. Total duration $T$ taken from demo length.
- **Robot model.** Kinematic-only chain of joint angles plus forward kinematics $\mathcal{K}(q)$; the high gear ratio (100:1) makes rope reaction torque negligible. PD joint servos provided by xArm.
- **Rope model.** 3D serial chain of $N=11$ point masses connected by fixed-length constraints with per-joint bending stiffness $k$ and damping $b$. End link mass $m_e$ is tunable. With $m=1$, the model has 5 free parameters $(k, b, m_e, l, N)$. Implemented as a maximal-coordinate variational integrator (Lavalle-Tedrake style). The first link is *kinematically driven* by the fingertip trajectory.
- **System model and linearization.** $\mathcal{M}(\mathbf{u}, \mathbf{z}_0) = f(\mathcal{K}(\mathcal{B}(\mathbf{u}, T)), \mathbf{z}_0) = \hat{\mathbf{x}}(t)$. Linearize $\partial \mathcal{M}/\partial \mathbf{u}$ about the current trial.
- **Inverse model = QP.** A Drake-formulated, Clarabel-solved quadratic program minimizes $\|\Delta \mathbf{x}(t_c) - \tilde{\mathbf{x}}_k(t_c)\|^2_{\mathbf{Q}}$ at the critical point, plus a fingertip-tracking cost on $t > t_c$ for follow-through, plus a control-effort cost $\mathbf{R}$ on arm joints. Constraints: linearized robot joint position/velocity/acceleration/torque bounds.
- **Initial guess.** Trajectory optimization (Drake + SNOPT) tracks the demo fingertip trajectory while satisfying joint dynamics — already enough to produce a runnable but unsuccessful command.
- **Algorithm.** Iterate: run the command on real hardware, measure rope state at $t_c$ via Vicon, solve the QP for $\Delta \mathbf{u}$, subtract it from the command, repeat. Stop on first success.
- **Evaluation hardware.** xArm 7 at 250Hz; 11 retroreflective markers per rope tracked by Vicon at 200Hz; 1.1m rope, weighted end.

## Results

- **Headline:** Task-Level ILC achieves **100% success within 10 trials on every one of 7 rope types** (chain, latex surgical tubing, braided/twisted ropes, 7-25mm thickness, 0.013-0.5 kg/m density). After convergence, the learned command is repeatable: 40-trial robustness test, zero failures.
- **Critical-point vs equal-weighted objective.** With equal weighting and the same model and demo, learning fails — error at $t_c$ stays large because update budget is spent reducing earlier-trajectory error that does not matter for the knot.
- **Demonstration-variation robustness.** Same algorithm learns successfully across 4 distinct flying-knot demos (durations 0.69-1.04s).
- **Cross-rope transfer.** Starting from a successful command on rope A, retraining on rope B converges in **2-5 trials** for most $(A, B)$ pairs; rope 7 (9mm latex) needs 0 additional trials from some sources. Some transfers (rope 5/6 → 2/3) fail to converge in 10 trials.
- **Model sensitivity.** Order-of-magnitude variation in stiffness $k$ and end mass $m_e$ leaves the success of learning intact for most settings — including a $10^4 \times$ stiffness sweep — confirming that ILC needs only correct *gradient direction* from the model, not accurate forward predictions.
- **Direct demo tracking is insufficient.** Following the human's hand trajectory alone (Initial Guess) does not produce a knot on rope 1 in any tested configuration, even though the human demo does — kinematic and dynamic limits of xArm 7 prevent exact reproduction.

## Limitations

- *Critical point is hand-annotated*, not auto-discovered. The whole approach hinges on knowing $t_c$ and that "match the rope shape at this instant" is task-relevant. Authors flag autonomous critical-point selection as the main open problem.
- *No collision modeling.* Rope-with-robot-body and rope-with-handle contacts are unmodeled and can drive the inverse model into unrecoverable regions.
- *Past-$t_c$ behavior is uncontrolled.* The objective is silent after the critical point, so a successful loop can untie itself in follow-through; lighter ropes can fail to tighten the knot.
- *Marker tracking degrades after self-contact.* Estimation of the rope's state at $t_c$ relies on a fixed-time critical point; if the rope contacts earlier than expected, the wrong frame is read.
- *Local minima and divergence.* Some transfer pairs (rope 5/6 → 2/3) fail to converge within 10 trials; cost can also increase after a poor update because there is no real-system line search.
- *Feedforward only.* The method learns $\mathbf{u}(t)$, not $\mathbf{u}(\mathbf{x})$. Unstable systems and large stochastic disturbances are out of scope.
- *Single-task evaluation.* Whipping, lassoing, and rope casting are mentioned as candidates, but only the flying knot is evaluated.

## Open questions

- Can the critical point be discovered automatically from a demonstration — e.g. via contact-state changes, dynamical extrema, or branch points in the rope trajectory?
- How does Task-Level ILC compare quantitatively to IRP / TossingBot / Real2Sim2Real on shared dynamic-rope benchmarks (when one is built)?
- Can the same critical-point principle be lifted into Behavior Cloning to make BC sample-efficient on dynamic deformable tasks?
- What is the right model-class taxonomy for "loose forward model, useful for ILC gradient" — when does the method break, in terms of model-eigenvalue alignment with the true system?
- Does ILC + critical-point work for cloth (2D), or do multiple critical points become necessary?

## My take

This paper is the conceptually-grown-up version of [[iterative-residual-policy-goal-conditioned-dynamic]]. IRP took the same observation — that dynamic rope tasks are repeatable, so trial-to-trial residual updates are the right unit of learning — and learned the residual *neural* policy from large-scale simulated data. This work strips out the simulator and the large data: one demonstration, a deliberately loose 11-link point-mass-bending model, a QP, and the closed loop runs on the real arm. The critical-point objective is the move that makes it work — without it, the same algorithm fails on the same hardware with the same demo, which is a clean ablation.

What I find most striking is the *robustness to model parameter error* (Table II): four orders of magnitude in stiffness and three orders in end mass leave success intact. That is consistent with the ILC theory — the inverse model only needs to provide a descent direction in command space — but seeing it survive on a contact-rich rope task is a stronger empirical claim than the theory carries on its own. It calls into question how much of the cost of "accurate DLO simulators" ([[deformx-versatile-co-simulation-framework-deformable]], [[deform-differentiable-discrete-elastic-rods-real]], [[rapid-adaptation-particle-dynamics-generalized-deformable]]) the field is paying for behavior that ILC could deliver from a much cheaper model. There is a real possibility that for a class of repeatable dynamic DLO tasks, the right move is not "build a better simulator" but "build a worse simulator and put a QP-ILC loop around the real robot."

The principal weakness is the manually-annotated critical point. In flying-knot the rope-self-contact frame is conspicuous, but for cloth flinging or whipping a target there are several plausible critical points and the choice matters. The hand-off problem — automatically picking $t_c$ — is also where this approach will or will not generalize beyond rope. Worth tracking what comes next from this group.

## Related

- [[task-level-ilc-cross-rope-transfer-2-to-5-trials]]
- [[task-level-ilc-real-hardware-flying-knot-100pct-under-10-trials]]
- [[task-level-iterative-learning-control]] — the method introduced here; the paper's principal new concept
- [[critical-point-objective]] — the trajectory-weighting trick that makes Task-Level ILC work on deformables
- [[optimization-based-inverse-model]] — QP-form inverse model (Drake + Clarabel) used at each ILC step
- [[mass-spring-system]] — the simplified rope model is a 3D mass-spring chain with fixed-length constraints and bending stiffness
- [[iterative-residual-policy-goal-conditioned-dynamic]] — conceptual ancestor; IRP learns the residual via deep nets on simulated data; this work replaces it with a one-shot QP and skips simulation
- [[planar-robot-casting-real2sim2real-self-supervised]] — alternative path: fit a simulator from physical rollouts, then plan
- [[robots-lost-arc-self-supervised-learning]] — dynamic rope manipulation via large-scale self-supervised data; baseline contrast
- [[krishna-suresh]]
