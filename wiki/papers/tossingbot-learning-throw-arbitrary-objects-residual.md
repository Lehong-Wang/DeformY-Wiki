---
title: "TossingBot: Learning to Throw Arbitrary Objects with Residual Physics"
slug: "tossingbot-learning-throw-arbitrary-objects-residual"
arxiv: "1903.11239"
venue: "RSS 2019 (Best Systems Paper) / IEEE T-RO 2020"
year: 2019
tags: [robotics, manipulation, throwing, residual-physics, grasping, deep-learning, sim-to-real, self-supervised, pick-and-place]
importance: 5
date_added: 2026-05-06
source_type: tex
s2_id: "42bfe1fd848644059061ec4ee47f96af14ec6184"
keywords: [residual physics, throwing, pick-and-throw, grasping affordance, fully convolutional network, ballistic prior, hybrid controller, self-supervised manipulation, MPPH, UR5]
domain: "Robotics"
code_url: "http://tossingbot.cs.princeton.edu"
cited_by: []
---

## Problem

Throwing is dynamic extrinsic dexterity: it lets an arm place objects faster and beyond its kinematic reach. But precisely throwing *arbitrary objects* in unstructured settings is hard because performance depends on (i) the pre-throw grasp condition (e.g. grasping a screwdriver by tip vs. by handle changes its centripetal release velocity), (ii) heterogeneous object-centric properties (mass distribution, friction, shape), and (iii) aerodynamics. Prior throwing systems sidestep these by assuming homogeneous objects (balls/darts) at fixed pre-throw poses, manually reset between trials. Conversely, prior pick-and-place systems achieve at most ~300 mean picks per hour (MPPH) and cannot place outside reach. The paper asks: can a single robot, end-to-end, learn to grasp arbitrary objects from a cluttered bin *and* throw them accurately into target boxes outside its reach range?

## Key idea

Two coupled ideas:

1. **Joint grasping–throwing learning** with a fully convolutional network $f(I, p)$ that maps an RGB-D heightmap $I$ and a target landing position $p$ to dense pixel-wise predictions of (a) grasp success likelihood $Q_g$ and (b) release velocities $Q_t$, sharing a perception backbone. Grasping is supervised by **whether the throw lands**, not just by gripper-width grasp success — so the policy learns grasps that lead to predictable throws.
2. **Residual Physics**: instead of learning the throw release velocity from scratch or relying solely on an analytical ballistic controller, predict a learned residual $\delta$ on top of the velocity $\hat{v}$ given by closed-form projectile equations: $\|v\| = \|\hat v\| + \delta$. The analytical prior provides cheap generalization to new target locations; the residual compensates for grasp-conditioned offsets, contact dynamics, and aerodynamics that ballistics cannot model.

The architectural insight is that the residual is on the **action space** (control parameters), not on the predicted next state, and that this makes it broadly applicable as a hybrid policy template (Fig. *model-variants*: type d).

## Method

- **Input.** RGB-D heightmaps of the bin ($180\times140$ at 5 mm/pixel) plus target box position $p$. Heightmaps are rotated through 16 orientations to handle 16 grasp angles.
- **Perception.** 7-layer fully convolutional residual network with 2 max-pool stages, output $\mu \in \mathbb{R}^{45\times35\times512}$. The estimated physics velocity $\hat v$ is concatenated as a 128-channel constant tile so grasping/throwing predictions are conditioned on it.
- **Grasp module.** FCN head outputs pixel-wise grasp success probability $Q_g$. The argmax pixel + heightmap rotation gives parallel-jaw grasp parameters $\phi_g = (x, \theta)$ for an open-loop top-down primitive (approach, close, lift 10 cm).
- **Throw module.** FCN head outputs pixel-wise residual $\delta_i$. Release position $r$ is fixed via two geometric constraints (release-distance from base $c_d$ at fixed height $c_h$, release-velocity direction angled $45°$ upward), reducing throw planning to a single scalar magnitude $\|v_{x,y}\|$. Residual is added to the analytical $\|\hat v_{x,y}\|$ obtained by inverting $p = r + \hat v t + \tfrac12 a t^2$ assuming point-mass ballistic motion.
- **Throwing primitive.** Robot curls inward while holding the object, uncurls at high speed, opens gripper at planned $r, v$. Gripper orientation kept orthogonal to the throw plane.
- **Loss.** $\mathcal L = \mathcal L_g + y_i \mathcal L_t$, with binary cross-entropy on the executed grasp pixel and Huber loss on the residual; gradient flows only through the executed pixel.
- **Self-supervised training.** Online trial-and-error on a UR5 with RG2 gripper. Ground-truth $y_i$ has two flavors — "object in gripper after grasp" and "throw landed in correct box" — and the latter (supervising grasps by *throws*) yields better throws and (slightly) better grasps. Ground-truth residual $\bar\delta_i = \|v_{x,y}\| - \|\hat v_{x,y}\|_{\hat p}$ is computed from the actual landing $\hat p$ tracked by an overhead RealSense. Prioritized experience replay; $\epsilon$-greedy decayed $0.5\to0.1$; SGD lr $10^{-4}$, momentum 0.9, weight decay $2^{-5}$. PyBullet simulation for ablations; real-world reset by tilting the landing zone so objects slide back into the bin (minimal human intervention).
- **System pipeline.** Asynchronous: training, inference, grasp execution, and throw execution overlap to push throughput.

## Results

- **Real-world throwing accuracy.** Residual-physics: 84.7% / 82.3% on seen / unseen objects. Beats Physics-only (61.3 / 58.5%) and Regression-PoP (54.2 / 52.0%). Roughly matches an untrained 15-person human baseline (80.1 ± 10.8%).
- **Real-world grasping success.** ~87% / 73% (seen / unseen). Comparable across baselines; the gain from residual physics is mostly in throwing.
- **Throughput.** 514 MPPH (608 grasps/h × 84.7% throw accuracy) on a UR5. State-of-the-art before TossingBot: Cartman 120, Dex-Net 2.0 250, FC-GQ-CNN 296, Dex-Net 4.0 312 MPPH. TossingBot-with-placing (no throw) is 432 MPPH.
- **Generalization to new target boxes.** Residual-physics 87.2% / 83.9% (sim / real) on unseen target locations vs. Regression-PoP at 26.5 / 32.7% — direct empirical evidence that the analytical prior provides the generalization while the residual provides the precision.
- **Simulation ablations.** Residual-physics > Physics-only > Regression-PoP > Regression on hammers, rods, cubes, balls, seen, and unseen sets; gap widens on hard objects (hammers: 81.2% vs. 70.4% vs. 47.8% vs. 32.8%).
- **Grasp-by-throw supervision.** Supervising grasps with throw success produces a more spatially restricted but more stable grasp distribution (concentrated slightly off the CoM, away from object ends).
- **Emergent visual semantics.** The perception module's deep features cluster objects by shape and aerial-relevant physical attributes without any class supervision (e.g. picking a ping-pong-ball pixel highlights all other ping-pong balls, ignoring same-color blocks).

## Limitations

- **Rigid objects only.** Assumes objects can withstand throw forces; no story for fragile, articulated, or deformable items.
- **Vision-only sensing.** No tactile or force-torque feedback to refine grasp- and aerodynamic-induced corrections at release.
- **Position-only control of the projectile.** Cannot target a desired pose / orientation in flight (no in-flight orientation control).
- **Single-task scope.** Residual Physics is demonstrated only on throwing; whether the same residual-on-action-space recipe transfers to other contact-rich tasks is left open by the paper.
- **Two-stage open-loop primitive.** Grasp and throw are open-loop after planning; no mid-execution replanning.
- **Ballistic prior assumes point-mass.** Linear-trajectory and zero-cross-drag assumptions are baked in; while the residual mostly compensates, the tail of failure modes correlates with grasp configurations the residual cannot fix.

## Open questions

- Does Residual Physics generalize across tasks beyond throwing — e.g. striking, sliding, juggling, or DLO whip-control where dynamics models are partial?
- Can the residual be made *targeted-pose conditioned* (predicting orientation residuals jointly with velocity) to enable in-flight pose control?
- How would residual physics interact with deformable-object dynamics (cable tossing, soft-body launch), where the analytical prior is itself uncertain?
- What is the minimum data-efficiency a Residual Physics policy can reach if the analytical prior is poor (e.g. wrong drag model) — is there a graceful degradation curve back toward pure regression?
- Could the same architectural pattern wrap a *learned* dynamics model rather than a closed-form analytical one, giving "residual learning on residual learning"?

## My take

TossingBot is the canonical reference for the residual-physics design pattern as applied to robotic *control* (action-space residuals on top of an analytic policy), as opposed to residual learning on next-state predictions. For DeformY's interest in dynamic DLO manipulation, this is the methodological analog: bootstrap from a structured analytical prior (e.g. a Cosserat or simplified rod model that can predict where the tip flies given a base motion), and let a learned residual compensate for the grasp-conditioned, contact-conditioned, and aerodynamic factors the analytical model misses. The empirical lesson — that the residual provides accuracy *and* the analytical prior provides cross-target generalization, neither in isolation — is the precondition for low-data sim-to-real of a swing/throw-style DLO controller. The grasp-supervised-by-throw insight is also transferable: in DLO manipulation, supervising base motion by *whether the rope tip hit the target* could likewise discover base motions that lead to predictable rope dynamics, even when the rope state itself is poorly observed.

Sibling work to keep in mind for downstream methodological positioning: IRP (Iterative Residual Policy, [[iterative-residual-policy-goal-conditioned-dynamic]] in this wiki's pipeline) generalizes the residual idea to *iterative* refinement of a goal-conditioned dynamic action; DEFORM ([[deform-differentiable-discrete-elastic-rods-real]]) and DIDP ([[implicit-physics-aware-policy-dynamic-manipulation]]) push physics-aware action priors into deformable-object whipping; ETH-WB-Throw ([[learning-accurate-whole-body-throwing-high]]) extends accurate throwing to whole-body humanoid control; Real2Sim2Real planar-casting ([[planar-robot-casting-real2sim2real-self-supervised]]) and the Lost-Arc / Free-End-Cable line ([[robots-lost-arc-self-supervised-learning]], [[self-supervised-learning-dynamic-planar-manipulation]]) port self-supervised dynamic manipulation to ropes and cables. SoftMimicGen ([[softmimicgen-data-generation-system-scalable-robot]]), DaXBench ([[daxbench-benchmarking-deformable-object-manipulation-differentiable]]), Wiggle&Go ([[wiggle-go-system-identification-zero-shot]]), RAPiD ([[rapid-adaptation-particle-dynamics-generalized-deformable]]), RopeDreamer ([[ropedreamer-kinematic-recurrent-state-space-model]]), and Self-Curriculum-MBRL ([[self-curriculum-model-based-reinforcement-learning]]) supply the broader sim/data tooling layer. None of these supply paper-paper edges in INIT MODE — they only frame the methodological neighborhood.

## Related

- [[dynamic-throwing-and-hitting]]
**Foundations used**
- [[grasping]] — pixel-wise grasp affordance prediction is the perception–action interface
- [[behavioral-cloning]] / [[imitation-learning]] — TossingBot is *not* this; the contrast clarifies why self-supervised trial-and-error matters here
- [[backpropagation]], [[gradient-descent]], [[regularization]], [[optimization]] — training stack
- [[sim-to-real-transfer]] — simulation ablations + real deployment
- [[domain-randomization]] — randomized object color/placement during training (light DR)
- [[forward-kinematics]], [[inverse-kinematics]] — release-position/velocity planning and IK execution of the throwing primitive
- [[contact-rich-manipulation]] — pick-and-throw with arbitrary geometry sits in this regime

**Concepts introduced**
- [[residual-physics]] — the residual-on-action-space hybrid controller pattern (paper coins the term)
- [[throwing-motion-primitive]] — parameterized end-effector trajectory ending in a planned release position and velocity
- [[physics-informed-action-prior]] — using an analytical model (here ballistics) to seed control parameters before any learning, conditioning the network on the prior's output

**Claims supported**
- [[residual-physics-improves-throw-accuracy]]

**People**
- [[andy-zeng]], Shuran Song, Johnny Lee, Alberto Rodriguez, Thomas Funkhouser

**Important referenced work** (not yet ingested in this wiki — candidates for follow-up `/ingest`)
- Mason & Lynch — analytical throwing dynamics; the prior-art baseline.
- Dex-Net 2.0 / Dex-Net 4.0 / FC-GQ-CNN / Cartman — pick-and-place throughput baselines.
- Concurrent residual RL: Johannink et al. "Residual Reinforcement Learning"; Silver et al. "Residual Policy Learning".
- Kloss et al., Ajay et al., Fazeli et al. — residual *state* models for planar pushing (the contrast: action residuals vs. state residuals).
