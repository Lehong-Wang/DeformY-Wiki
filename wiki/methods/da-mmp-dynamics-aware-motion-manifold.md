---
name: "DA-MMP (Dynamics-Aware Motion Manifold Primitives)"
slug: da-mmp-dynamics-aware-motion-manifold
type: architecture
tags: [motion-manifold, movement-primitives, autoencoder, flow-matching, classifier-free-guidance, variable-length-trajectory, via-point-primitive, radial-basis-functions, kinodynamic-planning, dynamic-manipulation, throwing, goal-conditioned, dynamics-gap]
source_papers:
- "[[da-mmp-learning-coordinated-accurate-throwing]]"
parent_methods:
- "[[mmp-parametric-curve-motion-manifold-primitives]]"
child_methods: []
realizes_concepts:
- "[[motion-manifold-primitives]]"
- "[[execution-outcome-conditioned-trajectory-generation]]"
- "[[planner-generated-motion-corpus]]"
date_updated: 2026-07-30
---

## Problem setting

Generate a **whole-arm, high-speed, open-loop trajectory** that makes a released object reach a commanded goal, when (a) the required motion is too coordinated to express in a hand-designed action parameterization, (b) planned and executed trajectories diverge substantially (control tracking error at high acceleration, gripper slip and release-timing jitter, aerodynamics), and (c) real-world trials are expensive — tens, not thousands.

Assumed available: a robot model good enough for kinematic/kinodynamic planning, a simulator, a fixed grasp, and a measurement of the outcome (where the object landed). Not assumed: demonstrations, an accurate dynamics model, a reward function, per-goal retries at deployment, or closed-loop feedback during execution.

Instantiated in [[da-mmp-learning-coordinated-accurate-throwing]] on ring tossing with a 6-DoF arm: throw a 0.075 m ring around a 0.005 m peg 1–2.5 m away.

## Mechanism

Four components. The first two produce a manifold of *feasible* motions; the last two produce a map from goal to *accurate* motion.

**1. Variable-length via-point parameterization.** Each joint trajectory becomes
$$q(s;\mathbf w) = \psi(s) + \mathbf w^\top \boldsymbol\phi(s),\qquad s\in[0,1],$$
with $\psi$ a cubic Hermite spline pinning $\{q(0), q(1), \dot q(0), \dot q(1)\}$ and $\boldsymbol\phi$ the $K{=}30$ normalized Gaussian RBFs multiplied by the gate $(s(1-s))^2$ so they vanish at both endpoints (plain RBFs oscillated in endpoint velocity — and endpoint velocity *is* the throw). Weights come from one least-squares fit over positions and velocities jointly, with the time-to-phase factor $\alpha = 1/L$ converting $\dot q$ to $dq/ds$.

The distinctive move: **the execution horizon $L$ is a component of the parameter vector**, $\mathbf p_\tau = (\mathbf w,\, q(1),\, \dot q(1),\, L)$. This is what lifts MMP off fixed-length trajectories. The reason it cannot be avoided by time-normalizing: rescaling a trajectory to a common length re-times it, which changes release velocity, which changes where the object goes.

**2. Autoencoded manifold from planner-generated data.** A deterministic autoencoder $\mathbf p_\tau \mapsto \mathbf z_\tau \in \mathbb R^{64}$ trained with plain L2 reconstruction on 90k planner-generated trajectories (see [[planner-generated-motion-corpus]]). No isometric regularization, no VAE. The manifold's job here is *feasibility regularization*: the ablation shows a generative model trained directly on $\mathbf p_\tau$ emits oscillatory, unexecutable joint profiles.

**3. Latent conditional flow matching on executed outcomes.** A flow-matching model over $\mathbf z_\tau$ with classifier-free guidance, loss
$$\mathcal L_{\mathrm{CFM}} = \mathbb E_{u,\mathbf z_\tau,\mathbf z_{\mathrm{noise}}}\big[\|v_\theta(\mathbf z(u), u, \mathbf c) - v^\star(u)\|_2^2\big],$$
$\mathbf z(u)$ the linear interpolant between Gaussian noise and the true latent. The conditioning $\mathbf c$ is the **measured landing point $(x_{\mathrm{exe}}, y_{\mathrm{exe}})$ of that executed trajectory** — one label per executed trial, not the commanded target (see [[execution-outcome-conditioned-trajectory-generation]]). This is the whole dynamics story: no residual, no system identification, no explicit gap model.

**4. Inference.** Feed the desired target $(x_T, y_T)$ into the conditioning slot, integrate $v_\theta$ over $u\in[0,1]$, decode $\mathbf z_\tau \to \mathbf p_\tau \to$ joint trajectory, execute open-loop. One sample. No verification.

The reusable design principle: *put feasibility in the curve family and the manifold, put dynamics in the conditioning labels, and let the two be trained on datasets three orders of magnitude apart in size.*

## Procedure

**Stage I — corpus (offline, simulation, ~90k plans):**
1. Sample a throw state on the analytically-constrained goal manifold (release pose fixed by grasp and target geometry; spin about the ring normal, $\omega_z\sim[1.5\pi,3\pi]$ rad/s; horizontal release velocity with magnitude from projectile motion). Reject by quick-rejection heuristic, IK, and collision check; redraw on failure.
2. Plan home→throw-state with DIMT-RRT ($N_{\mathrm{planning}}=80$), smooth with $N_{\mathrm{smoothing}}=100$ bounded-acceleration shortcut iterations, collision-check at $\Delta t_{\mathrm{col}} = 1/30$ s.
3. Execute the plan **twice in simulation**; discard unless the two landings agree within threshold.
4. Interpolate to $f_{\mathrm{ctrl}} = 240$ Hz; fit $\mathbf p_\tau$ by least squares.

**Stage II — manifold:**
5. Train the AE (3-layer MLPs [256, 512, 256], LeakyReLU, $d_z=64$), ≤30k epochs with early stopping, batch 256, Adam lr 1e-4, wd 1e-5. Inputs normalized by mean/std.

**Stage III — dynamics (60 trials):**
6. Generate and execute ~60 trajectories at targets spread over $[1.5, 2.0]$ m; measure each landing at $z = z_{\mathrm{cyl}}$.
7. Encode each executed trajectory to $\mathbf z_\tau$; train CFM (6-layer MLP [256, 512, 1024, 1024, 512, 256], Swish) with $\mathbf c$ = measured landing, classifier-free guidance, ≤20k epochs, batch 450, Adam lr 3e-4, wd 1e-6.

**Deployment:**
8. Perceive target $(x_T, y_T)$; sample noise; integrate the vector field with the midpoint method, step 0.001; decode; execute; open gripper at $t=L$ and hold end-effector twist for a settling horizon $h_{\mathrm{settle}}$.

## Assumptions

- Fixed, hand-designed grasp; the object's state is a rigid transform of the end-effector state until release.
- Start and end at the same joint configuration with zero initial velocity (human-inspired wind-up-and-return structure).
- The goal manifold is analytically derivable — projectile equations supply release-velocity magnitude, so the sampler is not searching blindly.
- Task symmetry reduces the goal to a scalar radial distance (arm's first joint is a revolute symmetry axis); the model still consumes 2-D $(x,y)$ to absorb misalignment.
- Outcome is measurable to useful precision at deployment (RealSense + Canny/ellipse fit here) and roughly stationary across the 60 trials and the deployment session.
- Simulator feasibility (PyBullet, joint acceleration limits $[12.5, 12.5, 12.5, 15, 15, 15]$) transfers well enough that a plan feasible in sim is executable on hardware.

## Limitations

- **Single-sample deployment.** No best-of-N, no verifier, no rejection — the generative model's distributional nature is unused and an outlier draw is a lost throw.
- **Outcome data is embodiment- and object-specific.** New ring, gripper, or arm invalidates the 60 trials. Only the 90k manifold survives, and only for the same arm.
- **2-D conditioning.** Landing position only; release orientation and spin are not controlled and the success criterion tolerates a non-horizontal ring.
- **No latent geometry control.** Plain L2 AE at $d_z = 64$; the IMMP++ isometric-regularization lesson from [[mmp-parametric-curve-motion-manifold-primitives]] is not applied, and latent distortion is unmeasured — plausibly costly at 60 conditioning samples.
- **Corpus cost.** 90k kinodynamic plans, each with smoothing and a double simulated execution.
- **Reported real-world number is from a build with a known parameterization bug** (dropped $O(1)$ time-to-phase coefficient); the corrected version was only measured in simulation.
- **No public code or project page** as of 2026-07.

## Tradeoff profile

- **Real-world accuracy**: 60.0% ring-toss success vs 56.7% for humans given the same 60 practice trials, 13.3% for one planned attempt, 6.7% for residual-style correction. Small-$n$ (30 trials/condition), single-task evidence.
- **Real-data efficiency**: 60 executed trials — the headline. Bought by putting all the representation learning in simulation and only the dynamics in the real world.
- **Simulation/compute cost**: high and front-loaded — 90k plans + two AE/CFM trainings (≤30k and ≤20k epochs). Amortized across all goals.
- **Robustness to modeling error**: unusually high, demonstrated accidentally — a systematic parameterization bug corrupting every executed trajectory did not break the method, because conditioning labels are measured downstream of it.
- **Robustness to outcome *variance***: this is the axis where it beats residual approaches (which cancel bias only), and the reason its sim/real ordering inverts against them.
- **Inference cost**: one ODE integration over a 64-D latent with step 0.001 plus a decoder pass — milliseconds; no planning at deployment.
- **Extrapolation**: trained on $[1.5, 2.0]$ m, succeeds at 1.2 m; extent uncharacterized.
- **Smoothness**: structural, from the gated-RBF + Hermite curve family — MSSD ~280–296 vs ~556–596 for a waypoint parameterization at matched parameter count, and the gap does not close with more data.

## Evaluated by

- [[da-mmp-learning-coordinated-accurate-throwing]] — real and simulated ring tossing on a Galaxea A1 (6-DoF) with human novice/expert baselines, plus ablations on the autoencoder, corpus scale (0.09k–90k), and RBF-vs-waypoint parameterization.
