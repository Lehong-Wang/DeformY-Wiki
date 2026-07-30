---
title: "DEFORM: Differentiable Discrete Elastic Rods for Real-Time Modeling of Deformable Linear Objects"
slug: "deform-differentiable-discrete-elastic-rods-real"
arxiv: "2406.05931"
venue: "CoRL"
year: 2024
tags: [DLO, deformable-linear-object, discrete-elastic-rods, differentiable-simulation, residual-learning, position-based-dynamics, sim-to-real, robot-learning, system-identification]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "8cd61060dbd73a5193b9efa679a80ad891ba177e"
keywords: [DLO, DER, differentiable physics, residual learning, PBD, momentum conservation, system identification, real-time, MoCap, ARMOUR]
domain: "Robotics"
code_url: "https://roahmlab.github.io/DEFORM/"
cited_by: []
---

## Problem

Modeling deformable linear objects (DLOs) — ropes, cables, wires — during **dynamic motion** over **long time horizons** ($>1$ s) is a bottleneck for robotic manipulation tasks like surgical suturing and wire-harness assembly. The competing failure modes:

- **Pure physics-based models** (mass-spring, PBD/XPBD, FEM, DER) trade off compute vs. accuracy. PBD assumes quasi-static behavior and fails on dynamic swinging; FEM is too slow; mass-spring imposes artificial stiffness for inextensibility; standard DER lacks gradients and accumulates discrete-integration error.
- **Pure learning models** (Bi-LSTM, GNN) need massive real-world datasets, generalize poorly to new DLOs, and operate mostly in 2-D.
- **Differentiable cloth/DLO simulators** (DiffCloth, XPBD) tend to be numerically unstable when used for parameter ID on dynamic 3-D DLOs.

What is needed: a **dynamic** 3-D DLO model that runs in **real time**, has **gradients** for system identification + multi-step training, and stays **stable** even with stiff inextensibility constraints.

## Key idea

Wrap **Discrete Elastic Rods** (DER) into a differentiable simulator (**DDER**), then add two surgical fixes that DER alone cannot provide:

1. A **DNN residual** correcting the velocity/position update at each integration step — physics-grounded shortcut connection plus learned correction, trained end-to-end via the differentiable DER backbone.
2. A **mass-weighted PBD inextensibility correction** that, unlike vanilla PBD, **preserves linear and angular momentum**, so swinging ropes do not artificially decelerate or rotate during the constraint projection.

Result: a hybrid physics + learning DLO simulator that runs at 100 Hz, beats pure DER, pure Bi-LSTM, GNN, XPBD, and DRM on real-world dynamic prediction, and slots into a closed-loop perception + planning pipeline.

## Method

DEFORM is a four-part contribution stack:

**1. DDER (Differentiable Discrete Elastic Rods).**
Reimplement DER in PyTorch using autodiff. The single-step rollout requires solving an inner optimization for the per-edge twist $\bm{\theta}^*_t = \arg\min_{\bm{\theta}_t} P(\textbf{X}_t, \bm{\theta}_t, \bm{\alpha})$ that satisfies the quasi-static twist condition. Identifying material parameters $\bm{\alpha}$ (mass, bending/twisting stiffness) by minimizing a multi-step prediction loss therefore becomes a **bi-level optimization**, solved with implicit differentiation via Theseus.

**2. Residual integration.**
Inside semi-implicit Euler, inject a DNN residual on both the velocity and position update:
$$\hat{\textbf{V}}_{t+1} = \hat{\textbf{V}}_t - \Delta_t M^{-1}\left(\partial P/\partial \textbf{X}_t + \mathrm{DNN}(\hat{\textbf{X}}_t, \bm{\alpha})\right)$$
$$\hat{\textbf{X}}_{t+1} = \hat{\textbf{X}}_t + \Delta_t\left(\hat{\textbf{V}}_{t+1} + \mathrm{DNN}(\hat{\textbf{X}}_t, \hat{\textbf{V}}_t, \bm{\alpha})\right)$$
The DDER physics path is the ResNet-style shortcut; the DNN learns the integration-error correction.

**3. Momentum-conserving inextensibility (modified PBD).**
For each edge, define $\mathrm{C}(\hat{\textbf{x}}^i, \hat{\textbf{x}}^{i+1}) = \big| \|\hat{\textbf{x}}^i - \hat{\textbf{x}}^{i+1}\| - \|\bar{\textbf{e}}_i\| \big|$ and apply paired corrections. To satisfy $\sum_i \textbf{M}^i \Delta \hat{\textbf{x}}^i = \textbf{0}$ and $\sum_i \textbf{r}^i \times \textbf{M}^i \Delta \hat{\textbf{x}}^i = \textbf{0}$ (linear and angular momentum), the correction is scaled by a mass ratio $\beta^i = \|\textbf{M}^{i+1}\| / (\|\textbf{M}^i\| + \|\textbf{M}^{i+1}\|)$. Proven to satisfy both momentum equations.

**4. Multi-step training.**
Because every step is differentiable, train end-to-end with an L1 loss over a 100-step (1 s) prediction horizon, gradients flowing back through the residual DNN, material parameters, and the inner optimizer.

**Hardware.** Two industrial arms (Franka FR3 + Kinova Gen3), OptiTrack MoCap, RGB-D camera. 5 DLOs × 350 s of dynamic data each, 100 Hz. Train 80% / eval 20%, evaluation over a 500-step (5 s) horizon without any ground-truth feedback. Closed-loop integration uses ARMOUR for receding-horizon shape-matching planning.

## Results

**Modeling accuracy** (avg L1 over 5 s, $10^{-2}$ m, lower is better; 5 DLOs):

| Method | DLO 1 | DLO 2 | DLO 3 | DLO 4 | DLO 5 |
|---|---|---|---|---|---|
| XPBD | 4.00 | 3.85 | 3.80 | 3.62 | 4.35 |
| DRM | 3.64 | 4.16 | 3.21 | 4.02 | 4.19 |
| Bi-LSTM | 1.98 | 1.75 | 1.10 | 1.75 | 1.88 |
| GNN | 3.41 | 2.23 | 2.15 | 2.49 | 2.68 |
| DER (raw) | 1.96 | 1.91 | 1.86 | 1.50 | 1.65 |
| **DEFORM** | **1.01** | **0.97** | **0.77** | **0.85** | **0.99** |

DEFORM cuts 5 s prediction error roughly in half vs. the next-best (raw DER or Bi-LSTM) on every DLO.

**Speed.** 0.92–0.97 × $10^{-2}$ s per step (~100 Hz) — slower than Bi-LSTM (0.03 × $10^{-2}$ s) but real-time and real-time enough for closed-loop planning. Faster than raw DER and XPBD-NN.

**Ablations** (DLO 1 / 2 / 3, accuracy in $10^{-2}$ m):
- Raw DER: 1.96 / 1.91 / 1.86
- DDER w/o residual learning: 1.54 / 1.77 / 1.42
- DDER w/o system ID: 1.21 / 1.32 / 1.09
- DDER w/ original (non-momentum-preserving) PBD inextensibility: 1.65 / 1.23 / 1.00
- Full DEFORM: 1.01 / 0.97 / 0.77

Residual learning dominates the gain; system ID is non-trivial; momentum-preserving inextensibility helps most on compliant DLOs (DLO 1) where swinging is dominant.

**State estimation under occlusion (Bi-LSTM vs. DEFORM, RGB-D, varying sensor frequency).**
DEFORM with **no** sensor updates beats Bi-LSTM with 1 fps updates. DEFORM at 30 fps is best overall, with much smaller error growth under heavy occlusion.

**3-D shape matching with ARMOUR.**
- Real (20 trials): Bi-LSTM 10/20, **DEFORM 17/20**
- PyBullet sim (100 trials): Bi-LSTM 78/100, **DEFORM 90/100**

A success is defined as every vertex within 0.05 m of target within 30 s.

## Limitations

- Each DLO is **trained separately** on 350 s of MoCap-supervised data — no cross-DLO transfer / zero-shot generalization shown. New DLO = new dataset + new model.
- MoCap markers are physically attached during data collection and evaluation; their inertial effect is folded in but the paper does not quantify how much performance would degrade on unmarkered DLOs.
- Inner bi-level optimization through Theseus is the speed bottleneck — the model is real-time, but only by roughly 1× margin (0.95 ms/step at 100 Hz target).
- **No self-contact** of the DLO (knots, loops) is modeled; only smooth dynamic motion.
- Closed-loop manipulation uses ARMOUR — a quite specific receding-horizon planner; how DEFORM performs as a learned dynamics model under e.g. MPPI / RL is untested.
- Comparison to differentiable cloth simulators is qualitative; a head-to-head against, e.g., DiffCloth-DLO or DaXBench rope is missing.
- The paper does not test whether DDER's gradients are useful for sim-to-real transfer of an RL policy (only for system ID + perception).

## Open questions

- Can the per-DLO trained model be **conditioned on physical parameters** (length, density, stiffness) so a single DDER+DNN generalizes to a family of cables?
- Does the mass-weighted momentum-preserving inextensibility correction generalize to **self-contacting** DER (knots, tangles), where successive-vertex constraint corrections overlap with contact-pair corrections?
- The residual DNN compensates for integration error — at how large a $\Delta t$ does the DNN start having to learn dynamics it does not see in training (i.e., where does the physics-residual decomposition break)?
- For dynamic tip-targeting (the active DeformY problem), would training DDER through the planner end-to-end (gradients of task loss → policy → DDER → material params) close more of the sim-to-real gap than calibrating DDER offline once?

## My take

DEFORM is the methodologically tightest entry in the dynamic-DLO simulation family I have ingested so far: it identifies three precise failure modes of vanilla DER for dynamic 3-D rope simulation (no gradients, integration drift, momentum-violating inextensibility) and addresses each with a **minimal, principled, learnable** fix. The momentum-preserving PBD scaling is the single cleanest piece — it is a textbook-clean proof that vanilla PBD breaks momentum and a one-line $\beta^i$ fix that re-derives momentum conservation, and it shows up most where DLO compliance is highest.

For **DeformY**, the relevant takeaways are:
1. **Differentiable DER is the right backbone** if we want to do system ID against MoCap-calibrated real ropes — this is exactly the calibration step in the [[deformx-versatile-co-simulation-framework-deformable]] pipeline, and DEFORM gives a recipe (multi-step L1 loss + Theseus implicit diff) that is already validated on 5 DLOs.
2. **Residual physics + DNN beats pure physics or pure learning** for dynamic DLO modeling at the tested horizon. DeformX's PyElastica engine could host a DEFORM-style residual on top of its discrete Cosserat formulation; this is a straightforward upgrade path.
3. The **ARMOUR + DEFORM** closed-loop result (17/20 real, 90/100 sim) is the kind of evidence DeformY needs to produce, but for **dynamic tip-targeting** rather than static shape-matching. The trick is that DEFORM does not exhibit transfer to *new* DLOs — DeformY's claim is harder if it requires the same single-DLO calibration assumption.
4. The paper does not exploit DDER's differentiability for **policy learning**. That is the open lane: differentiable rollouts → analytic policy gradient → tip-targeting RL with sample efficiency a pure-PPO-on-Isaac approach cannot match.

The ablation that matters most for downstream work is the residual-learning row (1.96 → 1.54 → 1.01 m) — it argues that the physics prior alone is not enough for dynamic accuracy, but neither is pure learning; it is the *combination*, end-to-end through a differentiable physics core, that wins. That structural prior generalizes well beyond DEFORM's rope scope.

## Related

- [[dynamic-deformable-object-simulation]]
- [[sim-to-real-and-rapid-adaptation]]
**Foundations used**
- [[discrete-elastic-rods]] — methodological core, extended into DDER
- [[cosserat-rod-theory]] — continuum theory underlying DER
- [[deformable-linear-object]] — object class
- [[position-based-dynamics]] — basis of the inextensibility correction (modified for momentum)
- [[mass-spring-system]] — discussed as a baseline alternative
- [[finite-element-method]] — discussed as too-slow alternative
- [[sim-to-real-transfer]] — the empirical regime
- [[backpropagation]] — bi-level autodiff via Theseus
- [[gradient-descent]] — training and parameter ID
- [[regularization]] — multi-step L1 loss

**Concepts introduced**
- [[differentiable-discrete-elastic-rods]] — DER reformulated for autodiff + bi-level system ID
- [[neural-residual-on-physics-model]] — ResNet-style DNN correction wrapped around a physics integrator
- [[momentum-preserving-pbd-inextensibility]] — mass-ratio scaling of the PBD position correction so that linear and angular momentum are preserved during the inextensibility projection

**Claims supported**
- [[differentiable-der-plus-residual-realtime-dlo]]

**Important referenced work** (siblings under ingestion in this batch — relations recorded as forward edges only in INIT MODE)
- [[iterative-residual-policy-goal-conditioned-dynamic]] — residual physics for goal-conditioned dynamic DLO manipulation; complementary residual-learning paradigm at the policy level rather than the simulator level
- [[tossingbot-learning-throw-arbitrary-objects-residual]] — original residual-physics formulation in robotic manipulation; conceptual ancestor of DEFORM's residual integration
- [[planar-robot-casting-real2sim2real-self-supervised]] — alternative real2sim2real paradigm for DLO casting; complementary
- [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] — differentiable physics benchmark for deformables; same family as DEFORM but covers cloth/fluid/rope broadly
- [[accurate-simulation-parameter-identification-dlos-using]] — independent DER-in-MuJoCo system-ID work; closest direct competitor on calibration accuracy
- [[robots-lost-arc-self-supervised-learning]] — fixed-endpoint dynamic cable self-supervised learning; same problem family
- [[deformx-versatile-co-simulation-framework-deformable]] — Cosserat-Isaac co-simulation; DDER could plug in as the rod engine's residual-physics layer
- [[ropedreamer-kinematic-recurrent-state-space-model]] — recurrent latent dynamics for ropes; learning-based alternative to DEFORM's hybrid
- [[learning-deformable-object-manipulation-using-task]] — task-conditioned DLO manipulation learning
- [[implicit-physics-aware-policy-dynamic-manipulation]] — physics-informed policy for dynamic DLO manipulation
- [[learning-accurate-whole-body-throwing-high]] — ETH whole-body throwing; dynamic manipulation evidence
- [[dynamic-manipulation-deformable-objects-3d-simulation]] — flying-knot-ILC, ICRA dynamic manipulation
- [[softmimicgen-data-generation-system-scalable-robot]] — automated demonstration generation
- [[rapid-adaptation-particle-dynamics-generalized-deformable]] — RAPiD particle-dynamics adaptation
- [[self-curriculum-model-based-reinforcement-learning]] — model-based RL self-curriculum
- [[wiggle-go-system-identification-zero-shot]] — zero-shot system identification for dynamic deformables
- [[self-supervised-learning-dynamic-planar-manipulation]] — Free-end-cable / dynamic planar manipulation
