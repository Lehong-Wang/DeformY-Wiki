---
title: "Rapid Adaptation of Particle Dynamics for Generalized Deformable Object Mobile Manipulation"
slug: "rapid-adaptation-particle-dynamics-generalized-deformable"
arxiv: "2603.18246"
venue: ""
year: 2026
tags: [deformable-object-manipulation, mobile-manipulation, RMA, sim-to-real, visuomotor-policy, particle-dynamics, teacher-student, reinforcement-learning, adaptation, omnigibson]
importance: 3
date_added: 2026-05-06
source_type: tex
s2_id: "7d0762954ec8354ff712eeb623ceacb9f15d3164"
keywords: [RAPiD, Rapid Motor Adaptation, particle dynamics, deformable objects, mobile manipulation, dynamics embedding, shape embedding, privileged learning, teacher-student, OmniGibson, TIAGo, depth-image policy]
domain: "Robotics"
code_url: "https://sites.google.com/view/rapid-robotics"
cited_by: []
---

## Problem

Deformable objects — cables, ropes, cloths, hats, towels — react to manipulation in ways governed by **unknown dynamics parameters**: mass, position, but also material stiffness, friction, and the way the object's **shape** changes during interaction. The optimal action for covering a bowl with a stiff hat (pick-and-place) is qualitatively different from covering it with a soft cloth (fling, sweep, cover), and the agent does not know which regime it is in a priori.

Existing approaches each fail one piece of the requirement:

- **State-estimation methods** (particles, edges, graphs) need full object observability and break under self-occlusion or hand-eye coordination.
- **System identification** methods can recover dynamics parameters from real trajectories, but require multiple offline rollouts and so cannot adapt in real time.
- **Cross-embodiment imitation pretraining** (π0, OpenVLA, Octo, RDT) is data-hungry and does not zero-shot transfer to unusual embodiments such as a 22-DOF bimanual mobile manipulator.
- **Rapid Motor Adaptation (RMA)**, the natural framework, has been deployed for legged locomotion and in-hand rigid manipulation, but the dynamics it adapts to are mass and position alone — not the time-varying *shape* of a deformable object.

What is needed: a method that learns a real-time-adaptive visuomotor policy in simulation, transfers zero-shot to a real mobile manipulator under unseen object dynamics, categories, and instances, and adapts using **only** onboard depth and proprioception.

## Key idea

The recent **ground-truth particle positions** of a deformable object in simulation already encode shape change — this is the privileged signal that, in the rigid setting, RMA never needed because shape is a constant. Treat particle positions as the deformable analog of the privileged dynamics vector, then use the standard RMA two-phase teacher-student recipe to distill them into an adaptation module that runs from depth images alone.

Concretely, [[rma-particle-dynamics-adaptation]] separates the privileged input into two encoders — a **Shape Encoder** $\mu_s$ over recent particle positions and a **Dynamics Encoder** $\mu_d$ over object mass/position plus the shape embedding — and trains a visuomotor policy on top end-to-end with RL. Phase II then learns two adaptation modules ($\phi_s$, $\phi_d$) to predict the same embeddings from non-privileged depth + proprioception. At test time, encoders are discarded; only the policy and the two adaptation modules deploy.

## Method

**Setup.** RL is formulated as an MDP $\mathcal{M}=\langle\mathcal{S},\mathcal{O},\mathcal{A},\mathbb{T},\mathcal{R},\gamma,\rho_0,\mathcal{H}\rangle$ where $\mathcal{S}$ is observable only in simulation. Reward is binary task success plus a small dense distance shaping term. Training uses OmniGibson (BEHAVIOR-1K stack) on a simulated 22-DOF TIAGo bimanual mobile manipulator; depth at $224\times 224$, 3 Hz. Action space spans full DOF: 3-DOF omnidirectional base, 7-DOF $\times$ 2 arms, 1-DOF torso, 2-DOF head, 1-DOF $\times$ 2 grippers.

**Phase I — Privileged teacher.** Train end-to-end with RL:
1. **Shape Encoder** $\mu_s$: input is recent ground-truth particle positions of the deformable object plus recent robot actions; output is shape embedding $z_t^s$.
2. **Dynamics Encoder** $\mu_d$: input is ground-truth object mass and position plus $z_t^s$; output is dynamics embedding $z_t^d$.
3. **Visuomotor policy** $\pi$: input is current depth $o_t$ and $z_t^d$; output is full-DOF action.

The two-encoder split (rather than a single privileged head) is deliberate — it lets Phase II measure shape-vs-dynamics adaptation losses separately, which the ablations later show is load-bearing.

**Phase II — Non-privileged student.** Replace each encoder with an adaptation module trained by L1 regression against the frozen Phase-I embeddings:
- **Shape Adaptation** $\phi_s$: input is the last 10 depth images, joint angles, and actions; target is $z_t^s$.
- **Dynamics Adaptation** $\phi_d$: same input shape, target is $z_t^d$.
- Stop-gradient on the upstream RL signal into both modules so they specialize on shape vs. dynamics separately, and stop-gradient on $\phi_d$'s gradient flow into $\phi_s$ to prevent the shape module from re-encoding redundant dynamics.
- Then fine-tune $\pi$ on the inferred embeddings $\hat z_t^s$, $\hat z_t^d$ with RL.

**Test-time deployment.** The robot stores a 10-step buffer of (depth, action, joint angles), feeds it into $\phi_s$ and $\phi_d$, and updates the embeddings every 5 timesteps. The visuomotor policy consumes the current depth and current $\hat z_t^d$, outputting full-DOF actions. No QR codes, no real-robot demonstrations, no real-robot fine-tuning. The encoders from Phase I are discarded.

**Tasks.**
- `1D_Inserting`: pick up a 1D deformable object (rope, cable, belt, ribbon, ...) from the table and insert one end into a container; 20 unseen 1D categories at test time, 20 unseen container categories.
- `2D_Covering`: pick up a 2D deformable object (towel, cloth, plastic film, lid, ...) and cover ≥ 90% of a container's opening; 20 unseen 2D categories.

Each task is repeated 20 real-world rollouts within a 300 s timeout.

## Results

**Real-world success (20 trials each, no human intervention, 22-DOF TIAGo).**

| Method | `1D_Inserting` | `2D_Covering` | Total |
|---|---|---|---|
| **RAPiD** | 17/20 (85%) | 16/20 (80%) | 33/40 (82.5%) |
| DMfD | 3/20 (15%) | 1/20 (5%) | 4/40 (10%) |
| DDOD | 2/20 (10%) | 5/20 (25%) | 7/40 (17.5%) |
| RAPiD-No-Adapt | 7/20 (35%) | 5/20 (25%) | 12/40 (30%) |
| RAPiD-No-Shape | 7/20 (35%) | 9/20 (45%) | 16/40 (40%) |
| RAPiD-E2E | 5/20 (25%) | 4/20 (20%) | 9/40 (22.5%) |

RAPiD beats DMfD by 72.5 pp and DDOD by 65 pp aggregate. Both baselines fail because either the segmentation/dense-descriptor predictor breaks under occlusion and oblique camera angles, or — when the predictor works — the policies cannot adapt to unknown dynamics on the fly.

**Ablations.** Removing both adaptation modules (No-Adapt) drops total success by 52.5 pp. Removing only the shape module (No-Shape) drops by 42.5 pp — the policy can grasp and approach but cannot execute the dynamic insertion/covering motion. Replacing two-phase L1 distillation with end-to-end RL on the same architecture (E2E) drops by 60 pp; the high-dimensional 10-frame depth history plus unstable RL gradients cause E2E to collapse onto using only the current frame and ignore the dynamics embedding.

**Embedding probe.** A single dimension of $\hat z_t^d$ is qualitatively correlated with object softness (visualization figure). On a special test object whose rigidity changes during rotation, the same dimension tracks the rigidity transition during the rollout. This is suggestive evidence that the dynamics adaptation module recovers a meaningful softness/stiffness coordinate from depth + actions alone.

**Generalization scope tested.** 20 unseen 1D + 20 unseen 2D real-world categories (belts, hammers, markers, bananas, HDMI/VGA/GPU adapters, jumping rope, ethernet, zip ties, rakes, mice, screwdrivers, sticks, rubber, velvet rope, anchor rope, garden rope, ribbons, fans; polyester bags, pants, towels, plastic bags, face masks, paper bags, gloves, shorts, caps, lids, socks, soft cuffs, cardboard, envelopes, pouches, wallets, eye-masks, t-shirts, sponges, wool hats); unseen lighting and table environments; no parameter randomization on object pose/stiffness/friction (achieved implicitly via instance randomization).

## Limitations

- **Two tasks, 40 real-world trials total.** The headline 80%+ rate is computed across 20 trials per task; it is a reasonable but not exhaustive sample for the strong "across diverse dynamics, instances, categories" claim. Variance bars are not reported.
- **No comparison to recent VLA / cross-embodiment policies** (π0, OpenVLA, RDT, Octo). The paper argues these are not directly comparable on TIAGo because they would need infeasible real-robot demonstration counts; this is a defensible omission but leaves the question open whether sufficient pretraining would beat RAPiD's sim-only pipeline at the same task.
- **Shape encoding requires a particle-based simulator.** The method assumes the ground-truth particle positions are available in simulation. Tasks where the deformable physics in OmniGibson are not a faithful proxy for the real-world material (very stiff fabrics, highly nonlinear elastomers, fluid-like cloths) would inherit that gap.
- **Adaptation cadence is fixed at every 5 timesteps.** Why 5 is not ablated; tasks with faster-changing dynamics (e.g. high-frequency dynamic manipulation, throwing, casting) might require denser updates that conflict with policy temporal stability.
- **Reward shaping is hand-designed** (binary success + distance to target). Generalization to tasks without such a clean distance proxy (knot tying, folding to a target shape) is not demonstrated.
- **Single robot embodiment.** All experiments are on TIAGo. The paper's RMA framing is platform-agnostic in principle, but the simulator-side pipeline — tasks, randomization, depth-image conditioning — is tied to this 22-DOF mobile manipulator.
- **Citation count = 0** (S2, queried May 2026; March 2026 preprint, no venue listed). Reception is too early to read.

## Open questions

- Does the same Shape/Dynamics split outperform a single privileged-information head on tasks where shape and dynamics are tightly coupled (e.g. knot-formation), or is the split only useful when the two factors decorrelate?
- How small can the privileged simulator be for the teacher to still produce a useful student? Is OmniGibson particle granularity necessary, or does a lower-fidelity mass-spring or Cosserat-rod simulator (cf. [[deformx-versatile-co-simulation-framework-deformable]]) suffice for tasks where the deformable is a slender rod?
- Could the dynamics adaptation module be co-trained with system-ID-from-real-trajectories signals (cf. Wiggle&Go zero-shot system ID, IRP iterative residual policy) to close the small remaining 15-20% sim-to-real gap?
- How does RAPiD compare to differentiable-physics-based policy gradients (cf. DEFORM, DaXBench) on the same tasks? RAPiD is model-free; differentiable-physics methods could in principle exploit gradients of the same particle dynamics for policy learning.
- Will the embedding's emergent softness coordinate persist (or improve) when the simulator's deformable physics is replaced with a physically-richer model (e.g. Cosserat for 1D, FEM or PBD for 2D)?

## My take

RAPiD is a clean **specialization** of the RMA recipe to the deformable-object case, where the right privileged signal turns out to be the simulator's particle positions plus the object's mass/position. The novelty is not the two-phase teacher-student (RMA, Kumar 2021), nor the visuomotor policy (decade of prior art), nor the OmniGibson-on-TIAGo stack — it is the recognition that **shape change is a first-class component of "dynamics"** for deformable objects and must be encoded *separately* from rigid-body parameters. The ablation that drops the Shape Adaptation module (No-Shape, 42.5 pp loss) is the empirical lynchpin: it is the difference between "approaches the bowl" and "covers the bowl."

For DeformY-style **dynamic 1D tip targeting**, RAPiD is more relevant as a **transferable architecture template** than as a direct competitor: the current paper's tasks (insert one end, cover a bowl) are quasi-static-to-mildly-dynamic, not high-acceleration whipping or casting. But the structural pattern — privileged particle positions + non-privileged depth+action history → student adaptation module — is exactly the kind of teacher-student split that DeformY would want for transferring a Cosserat-rod-trained policy to a real swinging rope. The interesting question is whether the **shape embedding** would, on a 1D dynamic task, recover a useful coordinate over rope-length-and-stiffness products; the embedding-probe figure is suggestive evidence that it would.

Two cautious notes: (1) The 65+ pp margin over baselines (DMfD, DDOD) is genuinely large but partly reflects that those baselines pre-date the modern depth-image / RL-from-pixels regime; the absent VLA comparisons matter for benchmarking rigor. (2) The paper does not report variance, calibration of the simulated dynamics distribution against the real one, nor any failure-mode taxonomy for the 17.5% of trials that fail. Until reception accumulates, treat the headline number as a strong existence proof rather than a confirmed SOTA.

For ΩmegaWiki, the paper's lasting contribution is the [[rma-particle-dynamics-adaptation]] concept and the [[rma-particle-rapid-real-world-success]] empirical claim it produces: a single template for "use the simulator's particle access to make RMA work for things that change shape."

## Related

**Foundations used**
- [[deformable-linear-object]] — the 1D object class on `1D_Inserting`
- [[sim-to-real-transfer]] — the empirical regime of all results
- [[domain-randomization]] — implicit randomization is via 20-category instance diversity
- [[visuomotor-policy]] — the deployed policy form
- [[behavioral-cloning]] — Phase II distillation is regression onto a privileged teacher (BC-shaped, even though L1 + RL-finetune)
- [[grasping]] — task subroutine inside both `1D_Inserting` and `2D_Covering`

**Concepts introduced**
- [[rma-particle-dynamics-adaptation]] — RMA-style two-phase teacher-student adaptation specialized for deformable objects via privileged-particle-position encoders

**Claims supported**
- [[rma-particle-rapid-real-world-success]] — RAPiD's RMA-with-particle-dynamics pipeline produces > 80% success on two real-world deformable mobile-manipulation tasks with unseen dynamics, categories, and instances

**Important referenced work** (siblings under ingestion in this batch — relations recorded as forward edges only in INIT MODE)
- [[iterative-residual-policy-goal-conditioned-dynamic]] — alternative residual-policy adaptation for goal-conditioned dynamic deformable manipulation
- [[tossingbot-learning-throw-arbitrary-objects-residual]] — original residual-physics framing for sim-to-real manipulation
- [[deform-differentiable-discrete-elastic-rods-real]] — differentiable DER + residual learning; complementary simulator-side pipeline
- [[accurate-simulation-parameter-identification-dlos-using]] — DER-in-MuJoCo system ID; alternative real-world dynamics inference
- [[planar-robot-casting-real2sim2real-self-supervised]] — real2sim2real for free-end-cable casting; alternative sim-to-real recipe
- [[dynamic-manipulation-deformable-objects-3d-simulation]] — 3D dynamic deformable manipulation benchmark (DIDP)
- [[learning-accurate-whole-body-throwing-high]] — ETH whole-body throwing with residual policy (rigid analog)
- [[implicit-physics-aware-policy-dynamic-manipulation]] — physics-aware policy for dynamic rigid-via-soft manipulation
- [[learning-deformable-object-manipulation-using-task]] — Task-Level ILC for dynamic deformable shaping (Suresh & Atkeson, CMU 2026)
- [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] — differentiable benchmark for deformables; possible substrate for RAPiD-style training
- [[softmimicgen-data-generation-system-scalable-robot]] — automated demonstration generation; complementary to RAPiD's sim-only training
- [[ropedreamer-kinematic-recurrent-state-space-model]] — recurrent latent dynamics for ropes
- [[self-curriculum-model-based-reinforcement-learning]] — model-based RL with self-curriculum for shape control
- [[wiggle-go-system-identification-zero-shot]] — zero-shot system identification for dynamic ropes
- [[robots-lost-arc-self-supervised-learning]] — fixed-endpoint cable self-supervised learning
- [[self-supervised-learning-dynamic-planar-manipulation]] — free-end cable dynamic manipulation
