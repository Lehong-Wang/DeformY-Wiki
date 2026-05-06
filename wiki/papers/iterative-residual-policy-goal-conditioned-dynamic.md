---
title: "Iterative Residual Policy: For Goal-Conditioned Dynamic Manipulation of Deformable Objects"
slug: "iterative-residual-policy-goal-conditioned-dynamic"
arxiv: "2203.00663"
venue: "RSS 2022 (Best Paper); IJRR 2024"
year: 2022
tags: [DLO, deformable-linear-object, dynamic-manipulation, sim-to-real, residual-learning, iterative-control, rope-whipping, cloth-placement, robot-learning, mujoco]
importance: 5
date_added: 2026-05-06
source_type: tex
s2_id: "2b91aa3417fbe20cb18eb99e3775e486c9c25695"
keywords: [iterative residual policy, delta dynamics, goal-conditioned manipulation, dynamic deformable manipulation, rope whipping, cloth placement, sim-to-real, online adaptation, action sampling, DeepLabV3+]
domain: "Robotics"
code_url: "https://irp.cs.columbia.edu"
cited_by: []
---

## Problem

Goal-conditioned **dynamic** manipulation of deformable objects — e.g., whipping a rope to hit a precise target point in 3D space, or swinging a cloth to land in a specified keypoint configuration — is hard for three coupled reasons:

1. **Complex dynamics.** High-speed actions on under-actuated, infinite-DoF objects produce non-linear trajectories that are sensitive to initial conditions and measurement noise. Classical optimal control fails because an accurate forward model is prohibitively expensive to obtain.
2. **Hard-to-estimate object properties.** Stiffness, density distribution, friction, aerodynamics — all material in the dynamics, none easy to identify and none easy to map to common simulator parameters. The result is a large sim-to-real gap that domain randomization alone does not close.
3. **Strict goal precision.** Prior dynamic-manipulation work on deformables (cloth unfolding, cable vaulting) tolerates loose goal specification. This paper requires hitting a specific 3D point with the rope tip, or matching 9 cloth keypoints — accuracy of a few cm is the success bar.

The paper's goal is a single learning framework that achieves cm-level accuracy on these tasks **trained only in simulation on a fixed robot**, and that generalizes zero-shot to real ropes, novel materials/lengths/mass distributions, and even modified robot embodiments.

## Key idea

Instead of learning a full forward dynamics model, learn a **delta dynamics network**: given an observed trajectory $T_i$ produced by action $a_i$, predict the trajectory $\hat{T}_i^j$ that would result from a small action perturbation $a_i + \delta a_i^j$. At test time, sample many delta actions, score each predicted trajectory against the goal, take the best step, repeat. The previously-observed trajectory $T_i$ already encodes everything the network needs to know about the specific object and conditions; the network only has to learn how trajectories *change* under small action perturbations, which generalizes far better than learning the trajectories themselves.

Two ideas are doing the work simultaneously:

- **Iterative action refinement** (an iterative-learning-control style outer loop) — the system never has to be right on the first try; it just needs to take steps in the right direction.
- **Delta dynamics over residual trajectories** — the network never models the entire dynamical system; it predicts only the *change* in trajectory induced by a *change* in action, conditioned on the observed trajectory.

## Method

**Action primitives.** A whipping primitive parameterized by $a = (v, J_2, J_3)$ (max angular velocity and target joint angles for a UR5 + 50 cm wooden extension, 2D in the Y-Z plane). For cloth, $a$ is 4-D (cubic-spline via-points + total duration).

**Trajectory representation.** Each observed rope-tip trajectory $T_i$ is rasterized into a $256\times256$ binary image projected to the Y-Z plane. For cloth, 9 keypoint trajectories are stacked channel-wise. Predicted trajectories are real-valued occupancy maps in $[0,1]$.

**Delta Dynamics Network.** DeepLabV3+ takes the trajectory image plus the delta action broadcast to $N_a$ image channels, and predicts the resulting trajectory image. Trained with binary cross-entropy + AdamW (lr=1e-3, wd=1e-6) on 54 million simulated rope trajectories (12×12 grid of length × density, 50³ actions × 3 perturbations) from MuJoCo (rope = 25 linked capsules; cloth = 13×13 grid of capsules).

**Adaptive action sampling.** At each iteration, sample $N_s = 128$ delta actions from $\mathcal{N}(0, \sigma^2 I)$ with $\sigma = 0.5 \times d_i$ where $d_i = \mathcal{D}(T_i, g)$ — i.e., shrink the search radius as you approach the goal. Each predicted trajectory is thresholded at $t = 0.2$ to make it binary, distance to goal is computed, and the argmin delta is executed next.

**Initial action.** For rope whipping, $a_0$ is the action that minimizes average distance to goal across all training ropes (computable offline). For cloth, a constant initial action that keeps all 9 keypoints observable in the first frame.

**Stop conditions.** Real-world: $d_{\text{stop}} = 2$ cm or $\text{max\_step} = 10$. Simulation: 16 iterations.

## Results

**Rope whipping (real UR5).** Mean tip-to-goal error in cm at iteration 9, with IRP achieving cm-level accuracy across 7 unseen real ropes spanning material, length, mass distribution, and aerodynamics:

| Rope | OptSim | AVG | iterLinear-9 | IRP-9 |
|---|---|---|---|---|
| base-rope-100 | 21.6 | 15.6 | 13.4 | **1.3** |
| base-rope-120 | 14.4 | 16.5 | 22.5 | **1.9** |
| base-rope-60 | 14.5 | 19.9 | 28.0 | **5.5** |
| knotted | 14.2 | 8.3 | 9.2 | **2.6** |
| thick-rope | 11.9 | 19.7 | 10.7 | **3.2** |
| long-cloth | 15.8 | 59.6 | 16.8 | **1.9** |
| bullwhip | 17.0 | 28.7 | 8.4 | **1.9** |

Even **OptSim** (an oracle that exhaustively searches in simulation with measured rope parameters) is dominated, demonstrating that closing the sim-to-real gap requires online adaptation, not just better simulation parameters.

**Robot embodiment swap.** Replacing the wooden extension with 40 cm or 60 cm sticks (vs. 50 cm trained) — IRP still converges to good targets (elink-40: 6.1 cm, elink-60: 3.8 cm at iteration 9). The system learns to *re-map* actions through observed trajectories.

**Online recovery from system change.** Mid-experiment, knots are tied into the rope at step 6 (changing length, density, mass distribution); IRP regains good performance within ~3 additional iterations.

**Simulation results.** On extrapolation rope parameters (out-of-distribution in length × density), IRP reaches 1.5 cm; on interpolation, 0.4 cm. SysID baselines (even with ground-truth rope parameters) plateau at ~2-5 cm because un-modeled effects (aerodynamics, floor collision) are not in the parameter set.

**Cloth placement.** 3.5 cm mean keypoint distance across 11 unseen goal configurations, beating DeltaReg and IterHeuristic baselines with lower variance.

**Headline numbers.** 1.8 cm sim accuracy, 2.6 cm real-world accuracy.

## Limitations

- **Action repeatability is assumed.** The delta dynamics formulation requires that the same action produce a similar trajectory each time. Tasks with intrinsic stochasticity, non-recoverable failures (e.g., dropping the rope), or environment changes between actions break the assumption.
- **Full observability of the manipuland.** The trajectory rasterization assumes the object is visible end-to-end through the action; cluttered scenes or self-occlusion violate this.
- **Open-loop within an iteration.** The action is parameterized as a 3-D primitive; mid-action visual feedback is not used. Closed-loop visuomotor control during the swing is left to future work.
- **2D action space.** The whipping primitive lives in the Y-Z plane; out-of-plane goals are reached only by base-joint rotation. True 3D dynamics are not learned.
- **Single-rope simulation training set.** Training distributes only across length and density; surface area, stiffness, and mass distribution are out-of-distribution at test time and must be absorbed by online iteration.
- **No theoretical convergence guarantee.** The method is empirical; conditions under which iterative refinement converges (vs. oscillates or diverges) are not characterized.

## Open questions

- Can the delta-dynamics formulation be extended to **partially observable** scenarios using temporal cues or memory across iterations?
- Does the iterative-residual idea generalize to other domains where action repeatability holds — articulated objects, granular media, fluids, soft bodies?
- What is the smallest training distribution (in object-parameter space) that still produces useful zero-shot transfer? Is the 12² grid × 50³ actions necessary, or does a much smaller, well-chosen set suffice?
- How does delta-dynamics interact with **closed-loop visuomotor control** mid-swing? Could one learn a fast inner-loop policy that uses the iteration-learned action as a trajectory anchor?
- Multi-modality of solutions: when several actions reach the same goal, IRP commits to one via greedy selection. Could explicit multi-modal modeling (diffusion, sampling-based MPC) yield more robust convergence in pathological cases?

## My take

This is the **anchor paper** of the goal-conditioned dynamic DLO manipulation line and unambiguously deserves importance=5: RSS 2022 Best Paper, journal-extended in IJRR 2024, and the empirical setup (UR5 + rope-tip-to-goal in cm) is the reference benchmark that subsequent papers (including DeformX) calibrate against.

The methodological move is subtle but powerful: instead of trying to *predict* the world from action (which is a hard generative problem), predict the *delta* from a current observation (which is a much easier, more localized inference problem). The observed trajectory $T_i$ acts as an implicit physical-system identifier, and the delta network only has to learn the *Jacobian-like sensitivity* of trajectories to small action perturbations. This is structurally close to learned Iterative Learning Control (ILC), but lifted from the trajectory-tracking setting (where the reference trajectory is given) to the goal-reaching setting (where it is not). That lift is the key contribution.

For the DeformY follow-on: IRP's action-space limitation (planar 3-DoF, parameterized swing) is exactly the gap DeformX wants to close with closed-loop full-3D PPO policies. IRP and DeformX are **complementary**, not competing — IRP's iteration-learning idea + DeformX's higher-fidelity Cosserat simulation could unlock real-time adaptive 3D dynamic DLO control in a way neither approach achieves alone. A natural DeformY direction: layer iterative residual refinement on top of a closed-loop visuomotor policy trained in DeformX, using residual feedback only at decision points rather than per-action.

The 6-step recovery from rope-knotting is a striking demonstration that the formulation is genuinely doing online adaptation, not memorizing. That property is what makes the method robust to the long tail of real-world conditions that simulation cannot anticipate.

## Related

**Foundations used**

- [[deformable-linear-object]] — object class (rope and cloth)
- [[sim-to-real-transfer]] — the empirical claim space; IRP closes the gap via online adaptation rather than randomization or sysid
- [[jacobian-based-control]] — delta dynamics is structurally a learned Jacobian of trajectories with respect to action perturbations
- [[model-predictive-control]] — sample-and-evaluate action selection at each iteration is one-step MPC with a learned model
- [[visuomotor-policy]] — visual feedback (rasterized tip trajectory) drives action refinement
- [[domain-randomization]] — implicit randomization via the (length, density) training grid
- [[mass-spring-system]] — MuJoCo's linked-capsule rope/cloth model (the simulator's DLO representation)

**Concepts introduced**

- [[iterative-residual-policy]] — the overall framework: iterative refinement of a parameterized action via learned delta dynamics + adaptive action sampling
- [[delta-dynamics-network]] — the specific architectural pattern: a CNN that predicts the trajectory under a small action perturbation from the previously-observed trajectory plus the delta-action

**Claims supported**

- [[irp-zero-shot-sim2real-rope-whipping]]
- [[delta-dynamics-generalizes-better-than-sysid]]
