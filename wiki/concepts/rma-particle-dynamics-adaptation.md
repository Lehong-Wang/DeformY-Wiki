---
title: "RMA Particle-Dynamics Adaptation"
aliases: ["privileged-particle dynamics encoder", "RMA for deformable objects", "particle-position teacher-student adaptation", "shape-and-dynamics adaptation modules", "RAPiD adaptation"]
tags: [RMA, deformable-object-manipulation, sim-to-real, teacher-student, particle-dynamics, visuomotor-policy, robot-learning]
maturity: emerging
key_papers: ["[[rapid-adaptation-particle-dynamics-generalized-deformable]]", "[[rma-rapid-motor-adaptation-legged-robots]]"]
first_introduced: "2026"
date_updated: 2026-06-16
related_concepts: ["[[online-meta-learned-dynamics-adaptation]]", "[[amortized-context-encoder-adaptation]]"]
parent_topic: "[[sim-to-real-and-rapid-adaptation]]"
---

## Definition

RMA Particle-Dynamics Adaptation is the specialization of [Rapid Motor Adaptation](https://arxiv.org/abs/2107.04034) to **deformable-object** manipulation. Where standard RMA encodes a privileged dynamics vector — mass, friction, position — and learns a non-privileged adaptation module to predict that vector from proprioception and recent actions, RMA Particle-Dynamics adds the deformable's **time-varying shape** as a first-class component of the privileged signal: ground-truth particle positions of the object in simulation, paired with recent actions, are encoded into a Shape Embedding $z_t^s$, which is consumed alongside the rigid-body parameters by a Dynamics Encoder $\mu_d$ to produce $z_t^d$. The visuomotor policy conditions on $z_t^d$.

Phase II then trains two adaptation modules — Shape Adaptation $\phi_s$ and Dynamics Adaptation $\phi_d$ — to predict the same embeddings from non-privileged inputs (recent depth images, joint angles, actions) by L1 regression. At test time the encoders are discarded and the policy + adaptation modules are deployed end-to-end.

## Intuition

Standard RMA assumes that the things the policy needs to adapt to are **constant** for the duration of a rollout: a leg's mass does not change as a quadruped walks, and an in-hand cube's friction does not change while the hand reorients it. RMA distills this constant into a latent vector and learns to recover it from the rollout history. For deformable objects this assumption breaks: the policy must adapt to *how the object is reshaping itself right now*, which means the privileged vector is no longer constant and a single mass-and-friction encoding does not suffice.

The insight: the simulator has direct access to particle positions, so it can reveal the shape's current state to the teacher. Distill that into the student via L1, and the student learns to read shape state from depth + actions — the same channel an embodied agent uses in the real world. The shape and dynamics splits keep the two factors disentangled so each adaptation module's loss is interpretable, and stop-gradients between them prevent shape from absorbing dynamics or vice versa.

## Formal notation

Let $X_t \in \mathbb{R}^{P \times 3}$ be the simulator's $P$-particle position matrix at time $t$, $a_t \in \mathcal{A}$ the action, $o_t$ the depth observation, $q_t$ joint angles, and $m, p$ the deformable's mass and centroid position.

**Phase I (privileged):**
$$z_t^s = \mu_s(X_{t-k:t}, a_{t-k:t}), \quad z_t^d = \mu_d(m, p, z_t^s),$$
$$a_t = \pi(o_t, z_t^d)$$
Train $\{\mu_s, \mu_d, \pi\}$ jointly with RL.

**Phase II (non-privileged):**
$$\hat z_t^s = \phi_s(o_{t-k:t}, q_{t-k:t}, a_{t-k:t}), \quad \hat z_t^d = \phi_d(o_{t-k:t}, q_{t-k:t}, a_{t-k:t})$$
Loss: $\mathcal{L} = \|\hat z_t^s - z_t^s\|_1 + \|\hat z_t^d - z_t^d\|_1$, with stop-gradient on $\phi_d$'s gradient flow into $\phi_s$. Then fine-tune $\pi$ on $(\hat z_t^s, \hat z_t^d)$ with RL while stop-gradienting upstream RL gradients into both adaptation modules.

**Test time:**
$$a_t = \pi(o_t, \hat z_t^d), \quad \hat z_t^d \text{ updated every } 5 \text{ timesteps.}$$

## Variants

- **Particle-only adaptation**: skip the explicit shape/dynamics split and feed only privileged particle positions into a single encoder. Simpler, but cannot decompose adaptation losses into shape vs. dynamics — and the RAPiD ablation suggests this loses 42.5 pp on deformable mobile-manipulation tasks.
- **End-to-end RL on the same architecture (no Phase I)**: skip the privileged teacher and train the full pipeline with RL only. The depth-history input is too high-dimensional and the RL gradients are too unstable; in RAPiD this collapses to using only the current frame, with 60 pp loss in success.
- **Adaptation cadence sweep**: update embeddings every $k$ timesteps; RAPiD uses $k=5$ for temporal stability of the visuomotor policy, but the optimal $k$ likely depends on task time-constants.
- **Particle-substitute encodings**: replace ground-truth particles with a coarser shape proxy (skeleton landmarks, level-set, or a learned shape autoencoder) when the simulator does not expose particles. Untested in RAPiD, but a natural substitute when a Cosserat-rod simulator (no particle representation) provides the teacher (cf. [[cosserat-isaac-cosimulation]]).

## Comparison

- vs. **plain RMA** (locomotion, in-hand): plain RMA adapts to constant rigid-body parameters; RMA-Particle adapts to the time-varying shape of a deformable object. The non-privileged adaptation module's input grows by the depth-image history.
- vs. **System identification + sim parameter tuning** (Chebotar et al. 2019, real2sim2real): system ID needs multiple offline real trajectories; RMA-Particle adapts in real time during a single rollout.
- vs. **State-estimation manipulation** (particle/edge/graph trackers): state estimation requires full object observability; RMA-Particle adaptation is robust to occlusion because it operates on a low-dimensional latent inferred from depth + actions, not on a full pose recovery.
- vs. **Cross-embodiment imitation pretraining** (π0, OpenVLA, Octo, RDT): RMA-Particle is sim-only and zero-shots to unusual embodiments; large pretrained policies need real-robot fine-tuning data the target embodiment may not have.

## When to use

- The task involves a deformable object whose dynamics — including shape change — meaningfully affect the optimal action.
- A simulator with particle-level (or particle-equivalent) ground truth is available, and instance/category randomization in simulation can cover the real-world distribution.
- The deployed robot has access to depth observations and proprioception but no privileged state, and real-robot data collection is infeasible.

Skip when: the task is purely quasi-static and a calibrated rigid-body or single-shape model suffices; the simulator does not expose the privileged shape state; the policy needs to adapt to non-deformable factors (e.g. external dynamics like wind) that the privileged signal does not capture.

## Known limitations

- Requires a particle-based simulator (or equivalent shape ground truth) — not a drop-in for general physics simulators that don't expose particle positions.
- Empirically validated only on quasi-static-to-mildly-dynamic tasks (insert end of rope into bowl, cover a bowl with cloth); high-acceleration dynamic tasks (whipping, casting, throwing) are not yet tested.
- Adaptation cadence is fixed (every 5 timesteps in RAPiD); no analysis of how this couples to task time-constants.
- Variance of the headline real-world success rate is not reported; the empirical evidence is a single 20-trial run per task on a single robot.

## Open problems

- Does the shape/dynamics decomposition generalize to settings where the two factors couple tightly (e.g. knot tying, where the topological state is shape and dynamics simultaneously)?
- What is the smallest privileged shape signal that still produces a useful student? Is full particle position needed, or do skeleton landmarks / lower-dimensional shape latents suffice?
- Can RMA-Particle and system-ID-from-real-trajectories be combined to close the residual sim-to-real gap, especially in regimes where simulator deformable physics deviate from reality?
- Does the emergent softness coordinate observed in the dynamics embedding generalize to other shape-dependent adaptation problems (cloth folding, fabric draping, rope knot tying)?

## Key papers

- [[rapid-adaptation-particle-dynamics-generalized-deformable]] — introduces the concept; demonstrates 80%+ real-world success on two TIAGo deformable-mobile-manipulation tasks across 20+20 unseen object categories.
- [[rma-rapid-motor-adaptation-legged-robots]] — the parent RMA method this concept specializes; supplies the two-phase privileged-teacher / history-regressing-student recipe (see [[amortized-context-encoder-adaptation]]).

## My understanding

RMA-Particle is the right way to think about *what to encode* when extending teacher-student sim-to-real to deformables: shape is dynamics, and the simulator is the only place where shape is cheaply observable. The shape/dynamics split is a useful interpretability lever (it lets you ablate which factor matters more), and the empirical answer in RAPiD is that **shape matters more than rigid-body dynamics** for deformable mobile manipulation — losing the shape adaptation module is more damaging than losing the entire two-phase pipeline's RL improvements. This is consistent with the broader story that deformable-object policies are bottlenecked on shape estimation and shape-conditioned action selection.

For a rope-targeting follow-on (DeformY-style), the cleanest application would substitute the OmniGibson particle teacher with a [[cosserat-isaac-cosimulation]] teacher that exposes centerline + frame state, then distill into a depth-image student. The structural pattern — privileged shape + non-privileged depth-history → adaptation module — is independent of the particle representation, and the Cosserat representation is more compact and physically tighter for slender rods.
