---
name: "RMA: Rapid Motor Adaptation (two-phase context-encoder adaptation)"
slug: "rma-rapid-motor-adaptation"
type: training
tags: [sim-to-real, online-adaptation, teacher-student, context-encoder, system-identification, reinforcement-learning, legged-locomotion, privileged-learning]
source_papers: ["[[rma-rapid-motor-adaptation-legged-robots]]"]
parent_methods: []
child_methods: []
realizes_concepts: ["[[amortized-context-encoder-adaptation]]"]
code_repo: "https://ashish-kmr.github.io/rma-legged-robots/"
date_updated: 2026-06-16
---

## Problem setting

A control policy must adapt **online, in a fraction of a second**, to an environment whose physical parameters (friction, payload, motor strength, terrain, material properties) are unknown at deployment and may change at run time — under three constraints that rule out prior approaches: (1) no real-world data may be collected for adaptation (the robot might fall and break before it can walk), (2) no simulation calibration or real-world fine-tuning is allowed, and (3) the deployment platform has limited onboard compute. The setting assumes a simulator that exposes the true environment configuration $e_t$ during training and whose randomized distribution covers deployment conditions. Originally posed for quadruped locomotion (Unitree A1), but the method is platform- and task-agnostic wherever a privileged-information simulator is available.

## Mechanism

RMA decomposes the controller into a **base policy** $\pi$ and an **adaptation module** $\phi$, trained in two phases.

**Phase 1 — privileged base policy.** An environment-factor encoder $\mu$ compresses the privileged configuration $e_t$ into a low-dimensional latent — the **extrinsics** $z_t = \mu(e_t)$ — and the base policy consumes it:
$$z_t = \mu(e_t), \qquad a_t = \pi(x_t, a_{t-1}, z_t).$$
$\mu$ and $\pi$ are trained *jointly* by model-free RL (PPO). The extrinsics encode only *how the behavior should change* for the given environment, not the named parameters — a deliberately under-specified, behavior-relevant projection.

**Phase 2 — adaptation module (the reusable core).** Freeze $\mu$ and $\pi$. Train $\phi$ to regress the same extrinsics from a short window of *non-privileged* proprioceptive history:
$$\hat z_t = \phi(x_{t-k:t-1},\, a_{t-k:t-1}), \qquad \min_\phi \ \mathrm{MSE}(\hat z_t,\, z_t),\ \ z_t = \mu(e_t).$$
Both inputs and target are available in simulation, so $\phi$ is trained by **supervised regression** — no real-world data. The justification for regressing $z_t$ rather than $\hat e_t$: predicting exact parameters is harder and unnecessary (parameters can covary with identical observable effects), and end-to-end training only requires that $\hat z_t$ induce the right action.

The defining design choices that make the method reusable:
- **Estimate the latent, not the parameters** (amortized, behavior-relevant system ID).
- **On-policy DAgger-style data** for $\phi$ (roll $\pi$ under the *current* $\hat z_t$, pair with ground-truth $z_t$) so $\phi$ is robust to off-expert states it will see at deployment.
- **Asynchronous decoupling** at deployment: $\phi$ (slow, processes history) and $\pi$ (fast) run at different rates with no shared clock, since the extrinsics change slowly relative to state — enabling deployment on cheap compute.

## Procedure

1. **Define $e_t$ and the latent size.** Enumerate the privileged factors the teacher may see (e.g. mass + position, per-motor strength, friction, local terrain height → $\mathbb{R}^{17}$) and pick $d_z$ (e.g. $z_t \in \mathbb{R}^8$).
2. **Phase 1 (RL).** Jointly train $\mu$ (MLP) and $\pi$ (MLP) with PPO over a randomized environment distribution (e.g. fractal terrain + randomized friction/mass/motor strength), using task rewards (RMA: bioenergetics-inspired work/impact penalties + forward velocity) and a penalty curriculum to avoid collapse. Re-sample parameters within an episode to stress fast adaptation.
3. **Phase 2 (supervised).** Freeze $\mu, \pi$. Initialize $\phi$ (RMA: per-step MLP embed → 3-layer 1-D CNN over time → linear head). Iterate: roll $\pi$ using $\hat z_t = \phi(\text{history})$, store (history, ground-truth $z_t$), minimize MSE; repeat until convergence (DAgger).
4. **Deploy.** Run $\phi$ to produce $\hat z_t$ from the last $k$ steps (RMA: $k=50 \approx 0.5$s, at 10 Hz) and $\pi$ to act (RMA: 100 Hz), asynchronously, with $\pi$ consuming the most recent $\hat z_t$. No fine-tuning. (Optional **calibrate-then-freeze** variant: infer $\hat z$ once from a short probe and hold it fixed for an open-loop motion.)

Representative RMA hyperparameters: base policy + encoder MLPs (hidden 128 / 256-128); $\phi$ 1-D CNN (channels 32, kernels 8/5/5, strides 4/1/1); PPO 15,000 iters / batch 80,000 (~24h, ~1.2B steps); $\phi$ 1,000 iters with Adam, lr 5e-4 (~3h, ~80M steps).

## Assumptions

- A simulator exposes the privileged configuration $e_t$ at training time and is differentiable-free (model-free RL suffices).
- The training environment distribution **encompasses** deployment conditions (the latent must interpolate, not extrapolate).
- The non-privileged deployment history (proprioception / depth + actions) is **informative** about $z_t$ — i.e. the commanded-vs-actual discrepancy reveals the environment.
- The extrinsics change **slowly** relative to the base-policy control rate (justifies asynchronous deployment).
- The relevant adaptation target is (approximately) **constant** over the inference window — RMA's locomotion factors are; time-varying targets (e.g. deformable shape) need the augmented variant.

## Limitations

- Proprioception-only in the original form: blind to obstacles/drops that the recent history cannot reveal; large unforeseeable perturbations cause failures. Adding exteroception is open.
- The content of $e_t$ and the latent dimension are hand-designed and unanalyzed; for non-locomotion settings this is the dominant modeling decision.
- Fixed history length and update cadence, tied to platform compute and the dynamics time-constant.
- Requires the teacher's privileged information to exist in simulation; tasks whose key hidden factors the simulator cannot expose (or cannot simulate faithfully) inherit that gap.
- Performance is upper-bounded by the privileged Expert; the student recovers most but not all of it (RMA: 73.5% vs 76.2% Expert in sim).

## Tradeoff profile

- **Test-time adaptation cost:** a single forward pass (amortized) — **0** real-world adaptation samples, vs minutes of rollouts for test-time latent optimization (AWR) and an inner gradient/MPC solve per step for meta-RL model re-fit.
- **Adaptation speed:** sub-second (RMA: < 1s; the cited AWR baseline needs 4–8 min).
- **Optimality vs robustness:** recovers near-Expert performance (within ~3 pp in RMA sim) while a pure domain-randomization policy stays conservative — the adaptation module is what buys back the optimality DR trades away.
- **Compute at deployment:** very low; asynchronous design runs the slow encoder and fast policy independently on cheap hardware.
- **Engineering cost:** front-loaded — requires a privileged-information simulator, a two-phase training pipeline, and an on-policy DAgger loop for the encoder.

## Evaluated by

- [[rma-rapid-motor-adaptation-legged-robots]] — Unitree A1 quadruped. Sim: 73.5% success vs 76.2% privileged Expert and 62.4% (Robust) / 56.5% (SysID) / 41.7% (AWR) / 52.1% (no-adapt), with 0 adaptation samples and a mid-episode-changing environment. Real (zero-shot, no fine-tuning): 100% on memory-foam mattress / uneven foam, carries 12 kg (100% body weight), 80% step-down 15 cm, 90% oily patch; outdoors 100% on sand/mud/vegetation, 70% downstairs, 80% on cement/pebble piles. Extrinsics components shift and persist on slip / payload events, evidencing meaningful latent recovery.
