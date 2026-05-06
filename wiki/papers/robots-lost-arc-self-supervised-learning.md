---
title: "Robots of the Lost Arc: Self-Supervised Learning to Dynamically Manipulate Fixed-Endpoint Cables"
slug: "robots-lost-arc-self-supervised-learning"
arxiv: "2011.04840"
venue: "ICRA"
year: 2021
tags: [DLO, dynamic-manipulation, self-supervised, behavior-cloning, apex-point, minimum-jerk, UR5, fixed-endpoint-cable]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "6203047af3fbd2fc00299f76133cab44543934ec"
keywords: ["dynamic cable manipulation", "self-supervised learning", "minimum-jerk QP", "apex-point parameterization", "vaulting", "knocking", "weaving"]
domain: "Robotics"
code_url: "https://sites.google.com/berkeley.edu/dynrope/home"
cited_by: []
---

## Problem

Dynamic manipulation of a fixed-endpoint cable is hard because the action space is an entire timed-joint-trajectory and the cable's deformation under high-speed motion is impossible to model exactly. Prior cable-manipulation work largely assumes quasi-static dynamics (knot tying, untangling, shape matching) and parameterizes actions as 2D pick-and-place. The few dynamic baselines (e.g., Yamakawa et al.'s high-speed knot tying) require free-end cables and gravity-negligible regimes. The paper asks: can a robot teach itself a single high-speed arm motion that solves three concrete fixed-endpoint dynamic tasks — vaulting a cable over an obstacle, knocking an object off a pedestal, and weaving a cable between three obstacles — across cables of different thickness and mass?

## Key idea

Reduce the action space from a full trajectory to a single 3D apex point. Given fixed start and end joint configurations (left and right of the workspace), a trajectory function $f_{\mathcal{A}_i} : \mathbb{R}^3 \rightarrow \mathcal{A}$ generates a complete kinematically-feasible arm motion that passes through a learned apex configuration. Inside $f_{\mathcal{A}_i}$, a quadratic program (QP) — repeatedly shrunk in horizon $H$ until infeasible, in the spirit of GOMP/DJ-GOMP — minimizes squared jerk subject to joint limits and the start/apex/end equality constraints. A self-supervised loop (INDy) bootstraps from a single base apex found in a Blender Featherstone simulator, then collects real-robot trials by perturbing that apex until success and behavior-cloning the resulting CNN policy. This converts an infinite-dimensional dynamic manipulation problem into a 3D regression target conditioned on an overhead RGB image, while leaving the dynamics opaque to the learner.

## Method

- **Action representation**: $a \in \mathbb{R}^3$ = the (base, shoulder, elbow) joint angles of the UR5 at the trajectory midpoint; the remaining three joints are tied linearly to these.
- **Trajectory generation $f_{\mathcal{A}_i}$**: discretize $H+1$ steps with timestep $h$; minimize $\sum_t \|\vec{a}_{t+1}-\vec{a}_t\|_Q^2$ subject to start/apex/end position constraints, zero start/end velocity, configuration/velocity/acceleration/jerk limits, and Newtonian dynamics between steps. Repeatedly decrement $H$ until infeasible to make the motion as fast as the robot allows. Task identity selects start and end configurations only.
- **INDy training**: starting from a base action $a_{\beta,i}$ found by random search in simulation (10 environments, 60 candidate apexes, pick highest success), the physical UR5 self-resets the cable with a taut pull and tries random perturbations of the base until success; successful (image, apex) pairs are appended to $\mathcal{D}$. After collecting $N$ samples per task, a ResNet-34 + MLP head is trained via behavior cloning with MSE on a multivariate Gaussian over $a$ using the reparameterization trick.
- **Reset trick**: the taut-pull reset before every trial is what makes the system reproducible enough for self-supervised perturbation search — the same trajectory yields the same end state to within a tight envelope.
- **Simulator**: Blender + BulletPhysics Featherstone capsule chain, hand-tuned spring/torsion/damping; only used to find $a_{\beta,1}$ for vaulting, never to train the deployed policy. Note the deliberate sim-to-base, real-for-policy split — the authors explicitly characterize cable sim-to-real for vaulting and find ~50% of real failures succeed in sim, so sim is not trustworthy for evaluation.

## Results

- 5 cables (5.5m 18-gauge orange, 5m 16-gauge white, 6.2m 14-gauge orange, 5.5m 22-gauge ethernet, 4.8m 12-gauge jump rope), 3 difficulty tiers based on obstacle distance to robot base ($\le$1.5m / 1.5-3m / $\ge$3m).
- Vaulting: 81.7% (learned) vs 51.7% (fixed apex) vs 66.7% (human task-specific apex), 60-trial / 3-tier eval.
- Knocking: 65.0% (learned) vs 36.7% / 56.7% baselines.
- Weaving: 60.0% (learned) vs 15.0% / 15.0% baselines — the most dramatic gain over both baselines.
- Cross-cable transfer: policy trained on the 18-gauge orange cable transfers to four other cables of similar length without retraining (success rates between 40-80% across tasks/cables in Table III).
- Training cost: 180 trials × 8-9 hours per task on the physical UR5.
- Sim-to-real: ~50% of real failures succeed in simulation under matched apex/obstacle settings, motivating the choice to collect data on the physical robot.

## Limitations

- Length sensitivity: trained on a 5.5m cable, the policy collapses on a 6.2m cable (20%/10%/10% across tasks) because the reset cannot eliminate slack near the wall and the action energy is insufficient to accelerate the longer mass.
- Out-of-distribution obstacles: 7/9 Tier-3 vaulting failures come from obstacle horizontal shifts exceeding the 1.5m training bound.
- The framework requires fixed start/end configurations and a single apex; there is no mechanism for closed-loop feedback or multi-segment motions.
- Data collection is not asymptotically efficient — when obstacle size/location changes, the random-perturbation search restarts and there is no off-policy reuse of previous failures.
- The simulator is too coarse for policy evaluation (50% real-failure → sim-success rate), bounding what self-supervised exploration in sim could ever produce.

## Open questions

- Can the same 3D apex parameterization extend to free-end dynamic cable manipulation (the authors explicitly flag this as future work)?
- Is there a mathematical relation between apex joint configuration and cable landing location that bypasses the CNN regressor?
- How should the parameterization grow when one apex is no longer enough — adding a second control point, conditioning the policy on cable length, or learning the start/end configurations jointly?
- Does the reset-pull invariance generalize to cables with significant bending stiffness (where the reset pose is no longer a fixed point of the system)?

## My take

This is the canonical reference for dimensionality reduction in dynamic linear-deformable manipulation: collapse the trajectory action space to a single 3D apex and let a min-jerk QP fill in everything else. The trick is that the QP is the inductive bias — it absorbs all of the kinematic feasibility, jerk regularity, and timing structure that a full-trajectory policy would have to learn. What remains for the network is a low-dimensional regression that is easy to train with very modest data (~180 real trials per task). The shared lineage with [[planar-robot-casting-real2sim2real-self-supervised]] is visible in the self-supervised loop and the AUTOLab pipeline; the same group later relaxes the fixed-endpoint assumption in [[self-supervised-learning-dynamic-planar-manipulation]] (the free-end successor), which inherits the apex-point parameterization but moves into 2D planar dynamics. Compared to [[tossingbot-learning-throw-arbitrary-objects-residual]], which regresses a 2D throw vector on top of a physics prior, INDy regresses a 3D point on top of a planning prior — the two papers are best read together as twin examples of "learn the low-dim action, let a planner expand it into a trajectory".

The key conceptual contribution is [[apex-point-trajectory-parameterization]] as the 3D analog of TossingBot's 2D throw regression for cables. The unsolved scaling question (why does the policy collapse at 6.2m?) is the most useful place a follow-up could probe — it is exactly the energy/repeatability boundary of this whole class of methods. See [[iterative-residual-policy-goal-conditioned-dynamic]] (IRP) and [[implicit-physics-aware-policy-dynamic-manipulation]] (IPA) for goal-conditioned and physics-aware extensions; [[learning-deformable-object-manipulation-using-task]] (DEFORM) and [[deform-differentiable-discrete-elastic-rods-real]] (DER-MuJoCo) for differentiable-physics counterpoints; [[dynamic-manipulation-deformable-objects-3d-simulation]] (DIDP), [[learning-accurate-whole-body-throwing-high]] (ETH-WB-Throw), and [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] for adjacent dynamic-manipulation lines; [[ropedreamer-kinematic-recurrent-state-space-model]] and [[self-curriculum-model-based-reinforcement-learning]] for model-based DLO directions.

## Related

- [[apex-point-trajectory-parameterization]] — concept introduced by this paper
- [[dynamic-cable-apex-point-arc-policy]] — claim supported by this paper
- [[harry-zhang]] — first author
- [[ken-goldberg]] — senior author
- [[jeffrey-ichnowski]] — co-author
- [[deformable-linear-object]] — foundation
- [[behavioral-cloning]] — foundation
- [[imitation-learning]] — foundation
- [[sim-to-real-transfer]] — foundation
