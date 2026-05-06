---
title: "Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration"
slug: "learning-accurate-whole-body-throwing-high"
arxiv: "2506.16986"
venue: "IROS"
year: 2025
tags: [throwing, whole-body-control, legged-manipulator, residual-policy, MPC, reinforcement-learning, sim-to-real, ANYmal, dynamic-manipulation, robust-control]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "8fa35c65ad1f14dc022aa6b06820f79c6f73c053"
keywords: [prehensile-throwing, high-frequency-residual-policy, pullback-tube-acceleration, backward-reachable-tube, ANYmal-D, DynaArm, PPO, end-effector-tracking, whole-body-loco-manipulation, IROS-2025]
domain: "Robotics"
code_url: ""
cited_by: []
---

## Problem

Prehensile whole-body throwing — grasping an object, accelerating it through coordinated leg/torso/arm motion, and releasing mid-flight — is harder than fixed-base throwing because (i) the legged base injects coupled, partially uncertain dynamics into the end-effector (EE) trajectory, and (ii) the **release timing** is itself uncertain (gripper finger drive is non-repeatable, deformable objects detach over a 50–100 ms window). Prior whole-body throwers either skip accuracy reporting on hardware (UMI-Whole-body, Munn et al.) or restrict the system to fixed-base manipulators (TossingBot, Werner et al.). Model-based tube-acceleration methods (Liu and Billard) give release-time robustness on a single arm but assume the arm has a sufficiently accurate tracking controller and use an *open-loop* tube — inappropriate when an 18-DoF legged manipulator's tracking error exceeds the validity region of a pre-planned tube.

## Key idea

A three-layer stack that decouples accuracy from robustness:

1. A **100 Hz nominal RL policy** that tracks a timed EE position+velocity+orientation reference for an 18-joint ANYmal-D + DynaArm system (whole-body PD targets via PPO).
2. A **400 Hz residual policy** trained on top of the frozen nominal policy — same observation set plus current decimation and vertical-acceleration target, outputs arm-joint-position offsets — that closes the EE-tracking gap at the rate of state estimation, not the rate of the nominal policy.
3. A **pullback tube acceleration optimizer** running at >1 kHz: a parametric convex program (Disciplined Parametrized Programming → CVXPYgen, ~0.4 ms solve) that, given the *measured* live EE state and the backward reachable tube (BRT) of valid release configurations, computes a constant tube acceleration that *pulls* the EE state into the BRT during the 100 ms release window — closing Liu and Billard's open-loop tube around live tracking error.

Decomposition: nominal policy gets to the right region fast; residual policy fixes high-frequency tracking error; pullback optimizer renders release timing uncertainty harmless given the residual stack's tracking precision.

## Method

### Whole-body EE state tracking (nominal policy)

- 18-joint legged manipulator (ANYmal-D + DynaArm + Robotiq 2f140), formulated as time-dependent EE state tracking on the **goal manifold** of object flight (Pekarovskiy 2013).
- Episode structure follows Ma et al. badminton work: multiple throwing targets per episode, 2 s preparation period, training in `legged_gym` with PPO (4500 iterations, 2457 h sim time on RTX 3080 Ti).
- Observations at 100 Hz: robot state, target command, throw-elapsed time, previous action.
- Action: PD targets for all 18 joints. Constant-velocity reference per throw to simplify learning; gripper orientations chosen to avoid finger occlusion of the throw direction.
- Standard sim-to-real techniques applied: actuator networks (Hwangbo et al.), domain randomization (Tobin et al.), observation noise, symmetry augmentation (Mittal et al.).

### High-frequency residual policy

- Frozen nominal weights; train residual on top.
- Operates at 400 Hz (matches state-estimation rate; highest real-time-feasible feedback frequency).
- Inputs: nominal observation set + current control decimation + EE vertical acceleration target. Output: arm-joint-position offsets added to nominal action.
- Reward = nominal task reward (with position/velocity references modified by the acceleration command) + action-scale penalty (keep residual close to nominal, exploitation > exploration).
- Separate hyperparameters tuned for higher-frequency dynamics; 1200 PPO iterations / 655 h sim.

### Robust release: pullback tube acceleration optimizer

The original tube acceleration of Liu and Billard (2024) is *open-loop*: given a planned nominal release state, the BRT of valid landing trajectories is computed offline and the arm tracks a constant joint-space acceleration during the 100 ms release window. The authors observe that this fails for legged manipulators because residual tracking error makes the planned BRT-membership stale at release time.

Their fix is a **closed-loop** convex program **Tube-CVX**:

```
min        | r_land(blue) - r_target |^2          (decision: a_tube)
s.t.       p_T  = p_EE + T v_EE                   (free response)
           v_T  = v_EE + T a_tube
           rdot_T = || v_T,xy ||_2,  zdot_T = v_T,z
           rdot   = || v_EE,xy ||_2, zdot   = v_EE,z
           r_land0 = Phi(p_T, v_EE, z_land)       (BRT flowmap)
           r_land  = r_land0 + Jac · ([rdot_T - rdot, zdot_T - zdot]^T)
           v_min ≤ v_T ≤ v_max
           a_min ≤ a_tube ≤ a_max
```

Reading it as a controller: the BRT is a connected set in $\mathbb{R}^4$ (Khalil Th. 3.5), so a closed-loop tube acceleration computed from the *current* EE measurement makes the BRT an attracting invariant set of the throwing dynamics. Mathematically, this turns the open-loop tube into a feedback policy whose linearized constraint set is convex in $a_{tube}$ — Disciplined Parametrized Programming (DPP) form, generated via CVXPYgen, **0.4 ms** solve (75× faster than the original DPP-incompatible formulation), runnable >1 kHz.

### Deployment

- Onboard 400 Hz state estimator; nominal policy decimated to 100 Hz; residual + tube optimizer at 400 Hz.
- AprilTag-based target localization. Nominal release velocity computed from a parabolic flight trajectory ignoring drag.
- Pullback optimizer implemented as an asynchronous ROS node feeding the latest acceleration target.

## Results

### Headline

- **Mean landing error**: 0.276 m at 6 m target, 0.429 m at 4 m target (40 throws). Nominal-only baseline: 0.685 m / 0.710 m. Roughly **49.5%** average error reduction from residual + tube.
- **Human comparison**: 25 student participants throwing a floorball at a 25×28 cm target placed 3–4 m away. Robot scored 71/125 (56.8% success at fixed 6 m/s); participants 19/125 (15.2%). Best individual student: 3/5 in one round.
- Successful throws of gift boxes, snowballs (slip + variable mass + deformability), and floorballs both indoors and outdoors at 5–7 m.

### Whole-body contribution (base motion)

5 throws at 10 m/s horizontal velocity, dynamics computed via Pinocchio inverse dynamics. Base contributes 51.7 N·m·s of angular momentum and 46.3 J of work prior to release. Compared to a tabletop manipulator running the same joint trajectory, the legged base provides **53.4% higher angular impulse**. Base stays largely stationary at low velocities and tilts as commanded velocity grows — the legged base is genuinely used.

### Pullback tube ablation (1500-condition simulation grid)

3 EE heights × 5 horizontal vels × 4 vertical vels × 5×5 perturbation ratios, 5 seeds, 2 m/s² Gaussian acceleration noise. Max landing error in the 50–100 ms detach window:

| Release motion command | Max landing error (cm) |
|---|---|
| Constant velocity | 96.8 ± 57.9 |
| Pullback tube @ 100 Hz | 38.1 ± 57.9 |
| Pullback tube @ 200 Hz | 34.9 ± 23.6 |
| Pullback tube @ 400 Hz | **31.1 ± 17.7** |

Monotonic improvement with frequency — the *real-time* property is load-bearing.

### Residual policy ablation (500-trial simulated throws, 7 m target, 7 m/s)

Tuned 400 Hz residual + pullback tube > 400 Hz residual alone > 100 Hz residual > nominal. Full method achieves a **6.17%** landing accuracy improvement over residual-only and **20.04%** over nominal in simulation.

### EE velocity tracking (hardware)

At 4–6 m/s no clear residual vs. nominal difference; at 7–10 m/s the residual reduces velocity tracking error. At 10 m/s, mean velocity error is **16.8% lower** with residual.

## Limitations

- The residual policy only tracks vertical-direction accelerations — horizontal acceleration response is delegated to the nominal policy. The authors flag this as a constraint on overall accuracy.
- The residual policy improvement on hardware is much smaller than its improvement in simulation, suggesting a residual-specific sim-to-real gap — possibly overfitting training-environment dynamics. This is partially papered over by the pullback optimizer, but the residual is not pulling its weight at the WBC/MPC reference-tracker level.
- Object inertia is assumed negligible relative to the reflected EE inertia. For heavier or off-axis loads this assumption breaks; the authors propose estimating inertial properties during the preparation phase.
- Some headline results were collected with partial implementations or earlier checkpoints due to timeline pressure (snowball tests; human-comparison demo).
- Throwing accuracy reported only at one indoor venue with AprilTag target localization; outdoor robustness shown qualitatively.

## Open questions

- Can the residual close the sim-to-real gap fully (it currently helps less on hardware than in simulation)? Probably needs better simulator dynamics and/or a feedback-shaped residual structure rather than a free MLP offset.
- Does the pullback optimizer extend cleanly to non-projectile flight dynamics (drag, spin, deformable objects mid-flight)? The BRT becomes implicitly defined and would need a learned Neural Event ODE (Liu and Billard, 2024).
- Is the architectural template — `low-rate nominal RL` + `high-rate residual RL` + `tube-acceleration QP` — transferable to other dynamic whole-body skills (throwing softer/heavier objects, whipping a flexible payload, hammering, kicking)?
- Could the residual policy be *partially differentiable* with respect to the tube QP, training the residual to minimize predicted tube cost rather than just tracking the nominal reference?

## My take

This is a clean architectural template that's likely to generalize beyond throwing. The crucial conceptual move is recognizing that a standard residual-policy stack (Silver 2018 / Johannink 2019) is one *frequency-fixed* layer too few for highly dynamic whole-body skills, and that the missing ingredient is a sub-millisecond model-based optimizer that is itself closed-loop. Reading the paper through the lens of dynamic deformable manipulation (the active DeformY direction): **whipping a free-end cable** to a target, **throwing knotted DLOs**, and **dynamic rope-tip striking** all share the same release-time uncertainty pathology that the pullback tube optimizer is engineered to absorb. The natural successor of TossingBot's residual physics — TossingBot replaced one scalar velocity correction with a learned residual; this paper replaces the entire mid-execution correction with a **continuous** high-frequency residual + a sub-ms tube QP that closes the loop on release timing.

Two transfer caveats worth flagging early. (i) The pullback tube relies on a tractable BRT — projectile dynamics here. For DLO-tip striking, the equivalent BRT is over a flexible-body flight phase with continuous coupling; you cannot close it in the same convex form without further structure. (ii) The residual's hardware vs. simulation gap is a warning that high-frequency residuals can quietly memorize training-time idiosyncrasies. For DeformY-style dynamic manipulation, this argues for residual policies trained against domain-randomized simulator + a held-out hardware probe early in training.

## Related

**Foundations used**
- [[model-predictive-control]] — the conceptual ancestor of pullback tube acceleration (constant-acceleration release window with receding-horizon convex resolves)
- [[model-based-reinforcement-learning]] — three-layer stack mixes model-free RL with a model-based optimizer (the QP)
- [[domain-randomization]] — applied to release timing, mass, gripper friction during nominal + residual training
- [[sim-to-real-transfer]] — actuator networks, observation noise, symmetry augmentation; the empirical claim space
- [[contact-rich-manipulation]] — gripper-object detachment is exactly a contact-uncertainty problem
- [[grasping]] — Robotiq 2f140 gripper, prehensile premise

**Concepts introduced**
- [[high-frequency-residual-policy]] — 400 Hz residual on top of frozen 100 Hz nominal
- [[pullback-tube-acceleration]] — closed-loop convex program over the BRT, sub-ms via DPP/CVXPYgen
- [[whole-body-prehensile-throwing]] — task formulation as time-dependent EE state tracking on the flight goal manifold

**Claims supported**
- [[hf-residual-tube-stack-enables-accurate-whole-body-throwing]]
- [[legged-base-contributes-major-angular-impulse-in-throwing]]

**Important referenced work** (not yet ingested — candidates for follow-up `/ingest`)
- TossingBot (Zeng et al., 2020) — residual physics for fixed-base throwing; this paper is the natural successor (TossingBot used a scalar residual on a learned ballistic prior; this paper generalizes to a continuous high-frequency residual + closed-loop tube QP).
- Tube Acceleration (Liu and Billard, 2024) — the open-loop predecessor of pullback tube acceleration.
- IRP (Iterative Residual Policy, Chi et al.) — same design family in the dynamic deformable manipulation regime.
- Werner et al. 2024 — RL throwing on a 12-ton excavator (large-scale dynamic throwing).
- Munn et al. 2024 — whole-body throwing without quantified hardware accuracy.
- Portela et al. 2024 — RL whole-body EE pose tracking on rough terrain (the precision-tracking ancestor on the same robot family).
- Pekarovskiy 2013 — goal-manifold formulation of throwing.
- legged_gym (Rudin et al., 2022) — training simulator.
- Hwangbo et al. 2019 — actuator networks (sim-to-real foundation).
