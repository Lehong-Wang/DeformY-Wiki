---
title: "Implicit Physics-aware Policy for Dynamic Manipulation of Rigid Objects via Soft Body Tools"
slug: "implicit-physics-aware-policy-dynamic-manipulation"
arxiv: "2502.05696"
venue: "ICRA"
year: 2025
tags: [robotics, deformable-linear-objects, dynamic-manipulation, soft-tool-use, system-identification, one-shot, sim-to-real, heterogeneous-system]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "fefe4678bac7dd2ad13a7f287de61b5cbff21428"
keywords: [implicit physics, system identification, soft tool, rope manipulation, rigid object transport, one-shot policy, casting, MuJoCo]
domain: "Robotics"
code_url: ""
cited_by: []
---

## Problem

Robot tool-use research has so far concentrated on rigid tools acting on rigid objects. When the *tool itself* is soft (a rope, a cable) and is used dynamically — e.g. casting a rope to fling a tied-on payload toward a distant, occluded target — three coupled difficulties arise:

1. **Heterogeneous system dynamics.** The DLO and the attached rigid payload form a heterogeneous mechanical system whose high-acceleration response is hard to model analytically and depends on rope length, rope mass, payload mass and geometry, and friction.
2. **Multi-object interaction with mode switches.** Real casts often pivot the rope against an obstacle (a wall edge), so the dynamics change discontinuously before vs. after contact.
3. **Unobservable physical properties.** Friction coefficients and payload mass cannot be read from images, yet they dominate the cast outcome.
4. **One-shot execution.** Iterative residual approaches such as [[iterative-residual-policy-goal-conditioned-dynamic]] and [[tossingbot-learning-throw-arbitrary-objects-residual]] need multiple trials per goal; for rescue, escape, or transport scenarios a single attempt is the operational requirement.

The paper asks whether a learned policy can **identify the relevant physics from a single short pre-action interaction** and then **emit one accurate goal-conditioned cast** in a never-before-seen environment.

## Key idea

Replace iterative residual refinement with a **two-stage implicit-physics-aware (IPA) policy** that identifies physics in a fixed, predefined probing motion and folds the resulting trajectory map directly into the action regressor.

- **SysID stage:** the robot executes a fixed predefined high-acceleration short-horizon action $\bar{a}$ with no obstacle contact. The PoI's resulting trajectory is binarised into an ego-centric top-down trajectory map $\bar{\tau} \in \{0,1\}^{m\times m}$. Because $\bar{a}$ is identical across environments, every difference in $\bar{\tau}$ is induced by environment physics — friction, rope length/mass, payload weight — without ever naming or labeling those quantities.
- **Action prediction stage:** a ResNet $\pi$ ingests a 5-channel pixel-aligned tensor $(\bar{a}, \bar{\tau}, d, s, \text{aux})$ — broadcast SysID action, SysID trajectory map, depth map, segmentation map, broadcast auxiliary parameters — and regresses a single scalar cruising velocity $\hat{vel}$ that defines a symmetric trapezoidal velocity profile $(q_s, q_g, vel, acc)$.

The whole pipeline is self-supervised: random environments are sampled in MuJoCo, SysID is run once per environment, then up to 25 cast actions per environment are tried with the velocity squeezed downward from 6.28 rad/s; only successful (object-lands-in-target-rectangle) episodes enter the dataset, paired with their environment's $(\bar{a}, \bar{\tau})$.

The implicit framing is the design lever: by *not* naming the latent physics parameters, the network is forced to encode whatever subspace of physics actually matters for predicting $vel$, side-stepping both the sim-to-real labeling cost of explicit identification and the per-trial inner-loop cost of residual policies.

## Method

- **Task:** "object transport" (O2T). A UR5e + Robotiq 2F-85 holds one end of a rope; the other end is tied to a rigid payload box resting on a tabletop. An obstacle wall partially occludes a distant target rectangle. The robot must rotate its base joint with a single trapezoidal velocity profile so that the payload, after pivoting around the obstacle, comes to rest within the target rectangle.
- **Action parametrisation:** $a = (q_s, q_g, vel, acc)$ where $q_s, q_g$ are start and end base-joint angles, $vel$ is cruising velocity, $acc$ is acceleration. The policy fixes $q_s = 3.14\,\text{rad}$ and $acc = 3.14\,\text{rad/s}^2$, and lets $q_g$ be set by the obstacle's top-edge position; only $vel$ is predicted.
- **SysID action:** fixed $\bar{a} = (3.14, 1.04, 3.14, 8)$. Predefined and identical across all environments and runs.
- **Input tensor:** five $m\times m$ channels — (1) broadcast $\bar{a}$, (2) trajectory map $\bar{\tau}$, (3) depth map $d$, (4) segmentation map $s \in \{0,\ldots,5\}$ (soft tool / interactive object / tool-connected object / PoI goal / agent / other), (5) broadcast auxiliary fixed-action parameters.
- **Network:** ResNet (CNN with skip connections, He et al. 2016 family), trained with MSE between $\hat{vel}$ and ground-truth $vel$ from successful samples. PyTorch-Lightning, AdamW, batch 96, 37 epochs on a single RTX 3090.
- **Domain randomisation:** friction coefficient, rope length / radius / mass, payload dimensions / mass, obstacle wall length / width / position. Per-environment: 1 SysID run + ≤25 cast attempts; failed (overshoot/undershoot/collision) episodes are discarded.
- **Simulator:** MuJoCo. Splits 0.8 / 0.1 / 0.1 train / val / test.

## Results

**Simulated O2T (50 test environments)**

| Method | SR ↑ | Pos-Diff (m) ↓ | Vel-Diff ↓ |
|---|---|---|---|
| RND (uniform sampler) | 4.4% | 1.028 ± 2.357 | 1.953 ± 1.324 |
| SQ-RND (3-trial squeezing-bounds sampler) | 20.0% | 1.070 ± 1.852 | 1.172 ± 1.312 |
| IPA w/o SysID | 16.7% | 0.914 ± 1.459 | 0.626 ± 0.433 |
| IPA w/o SysID + CPN | 20.0% | 0.877 ± 1.373 | 0.583 ± 0.428 |
| IPA + CPN | 40.5% | 0.371 ± 0.604 | 0.239 ± 0.180 |
| **IPA (ours)** | **72.5%** | **0.347 ± 0.356** | **0.163 ± 0.163** |

- IPA roughly **3.6× the success rate** of SQ-RND (a fair iterative-feedback baseline) and **>4× IPA w/o SysID**, isolating the SysID stage as the dominant lever.
- IPA outperforms IPA + CPN, suggesting the configuration prediction network — useful in prior model-predictive residual systems like DeRi-IGP, [[iterative-residual-policy-goal-conditioned-dynamic]], and DeRi-Bot — does not transfer to this heterogeneous high-speed regime; the authors flag this as worth further study.
- **SysID consistency.** At fixed friction (0.25 / 0.45 / 0.65), IPA-class methods keep Vel-Diff low while non-SysID baselines undershoot at high friction and overshoot at low friction, confirming that $\bar{\tau}$ does carry friction information.

**Real-world O2T (8 cases, no real-world training)**

- IPA: Pos-Diff 0.428 m, Vel-Diff 0.151, SR 62.5%.
- IPA w/o SysID: Pos-Diff 0.778 m, Vel-Diff 0.801, SR 25%.
- The simulation-to-reality gap is real but small (72.5% → 62.5%); friction was varied in real by placing the payload on a baking tray.

## Limitations

- **Fixed task formulation.** Only the *cruising velocity* is predicted; $q_s$, $acc$, and the robot's other 5 joints are hand-set. The policy cannot retarget by changing approach angle or acceleration, which would matter for any non-rotational casting action.
- **Single moving primitive.** The action class is one trapezoidal velocity profile from a fixed base-joint home. Casting through more complex multi-segment trajectories is out of scope.
- **Stationary environment.** Obstacles must be still during the cast; the segmentation map is consumed once.
- **Predefined SysID action.** $\bar{a}$ is hand-designed, not learned. If $\bar{a}$ does not excite the physics modes that matter for the test environment (e.g. very high friction where the predefined motion barely moves the payload), $\bar{\tau}$ degenerates.
- **No object-class generalisation.** Trained / tested on box-shaped payloads only; arbitrary geometries unaddressed.
- **CPN underperformance unexplained.** The paper notes that adding configuration prediction *hurt* IPA but did not investigate why; this is left as future work.
- **Small real-world test set (n=8).** SR 62.5% from 8 trials has wide confidence bounds.

## Open questions

- Can $\bar{a}$ itself be **learned**, e.g. via an information-theoretic objective that maximises mutual information between $\bar{\tau}$ and the unobservable physics parameters most predictive of $vel$?
- Does the implicit-physics encoding **transfer across tasks**? E.g. take $\bar{\tau}$ from object transport and reuse it in rope whipping, knotting, or DLO shape control.
- How does the approach scale to **multi-stage casts** where post-contact pivoting is required and the policy must emit a sequence of velocity profiles?
- Is there a **predictive variant of CPN** that *does* help here? IPA + CPN underperforms IPA, contradicting prior residual-policy intuition; this is worth a focused study.
- What is the **sample-efficiency curve** for the SysID stage as a function of $\bar{\tau}$ resolution $m$ and SysID horizon length?

## My take

IPA is the cleanest articulation in the DLO-as-tool literature of the trade-off between **iterative residual loops** (TossingBot, IRP) and **one-shot implicit physics encoding**. The contribution is not a new architecture (it's a small ResNet) but a *task formulation*: by fixing both ends of the action — the predefined SysID probe $\bar{a}$ and the analytical action parametrisation $(q_s, q_g, vel, acc)$ — the network's only job is to map $\bar{\tau} \to \Delta vel$. That makes the implicit-physics signal extremely tight, which is why a single MuJoCo-trained ResNet generalises to a baking-tray real-world friction shift without any real-world training.

The honest comparison is to [[iterative-residual-policy-goal-conditioned-dynamic]]. IRP iteratively refines an action over multiple trials per goal; IPA refines once across a SysID action then commits. IRP-style methods will dominate when (a) you can afford per-goal trials and (b) the system is roughly stationary across trials. IPA-style methods will dominate when (a) trials are scarce or destructive (rescue, escape, single-shot transport — exactly the motivating scenarios in the paper's introduction) and (b) physics shifts between goals (varying friction, varying payload). The 72.5% vs ~30% gap on O2T is consistent with that decomposition: one trial budget, heterogeneous physics across episodes.

Two things to watch:

1. The CPN result *contradicts* the residual-policy literature's intuition that a forward dynamics critic always helps. If reproduced, it suggests heterogeneous high-speed regimes break the assumption that CPNs cleanly disentangle action proposal from forward simulation. This is a small but real challenge to the [[iterative-residual-policy-goal-conditioned-dynamic]] / TossingBot residual-physics frame.
2. Implicit-physics encoders are a general primitive. The same SysID-then-commit pattern applies to flying-knot iterative learning control, whip cracking, free-end cable shaping, and the throwing-policy literature — each currently using its own iterative residual loop. IPA's one-shot framing is a candidate replacement.

The result is a single-paper preprint with n=8 real cases, so confidence on the headline claim is moderate, not high. But the *task setup* is distinctive enough that I expect this paper to anchor the "anti-IRP" position in DLO dynamic manipulation for the next ICRA / CoRL cycle.

## Related

- [[iterative-residual-policy-goal-conditioned-dynamic]] — the closest baseline in framing; IPA is explicitly designed as an anti-IRP one-shot alternative for heterogeneous systems.
- [[tossingbot-learning-throw-arbitrary-objects-residual]] — residual-physics one-shot throwing of *rigid* objects; same one-shot ambition, different physics regime.
- [[planar-robot-casting-real2sim2real-self-supervised]] — planar casting with similar moving primitive but no heterogeneous-system handling and re-training required for new physics.
- [[deform-differentiable-discrete-elastic-rods-real]] — explicit DLO physics modeling, complementary to IPA's implicit approach.
- [[accurate-simulation-parameter-identification-dlos-using]] — explicit per-DLO sysID via discrete elastic rods; the explicit counterpart to IPA's implicit sysID.
- [[learning-deformable-object-manipulation-using-task]] — task-level iterative learning control for DLO manipulation; iterative refinement family.
- [[robots-lost-arc-self-supervised-learning]] — self-supervised dynamic cable manipulation, fixed-endpoint variant.
- [[self-supervised-learning-dynamic-planar-manipulation]] — free-end cable analogue; complementary task class.
- [[wiggle-go-system-identification-zero-shot]] — explicit short-interaction sysID for zero-shot rope manipulation; closest non-implicit cousin.
- [[rapid-adaptation-particle-dynamics-generalized-deformable]] — adaptation-based deformable mobile manipulation; another non-residual adaptation route.
- [[ropedreamer-kinematic-recurrent-state-space-model]] — RSSM-based DLO dynamics learning.
- [[learning-accurate-whole-body-throwing-high]] — high-frequency residual throwing on legged platforms; comparison point for one-shot vs residual.
- [[daxbench-benchmarking-deformable-object-manipulation-differentiable]] — differentiable-physics benchmark for deformable manipulation.
- [[dynamic-manipulation-deformable-objects-3d-simulation]] — 3D dynamic-DLO benchmark; related sim platform.
- [[softmimicgen-data-generation-system-scalable-robot]] — scalable data generation for deformable manipulation; orthogonal data-side approach.
- [[implicit-system-identification]] — concept introduced/embodied here.
- [[heterogeneous-soft-rigid-system]] — concept the paper distinctively names and addresses.
- [[zixing-wang]] — first author.
- [[implicit-sysid-enables-one-shot-rope]] — claim supported by this paper.
- [[heterogeneous-payload-rope-dynamics-implicit-vs]] — claim weakly supported by this paper (vs. [[iterative-residual-policy-goal-conditioned-dynamic]]).
