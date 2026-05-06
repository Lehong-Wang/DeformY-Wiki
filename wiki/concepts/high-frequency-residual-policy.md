---
title: "High-frequency Residual Policy"
aliases: ["HF residual policy", "high-rate residual policy", "fast residual policy", "400 Hz residual policy"]
tags: [residual-policy, reinforcement-learning, whole-body-control, fast-feedback, sim-to-real]
maturity: emerging
key_papers: ["[[learning-accurate-whole-body-throwing-high]]"]
first_introduced: "2025"
date_updated: 2026-05-06
related_concepts: []
---

## Definition

A **high-frequency residual policy** is a learned correction layer trained on top of a *frozen* nominal policy that runs at a substantially higher control rate than the nominal — typically matching the rate of the underlying state estimator rather than the rate at which the nominal policy was trained. The residual observes the nominal policy's observation set plus high-rate quantities (current decimation index, fast-loop targets) and outputs *additive* offsets that are summed with the nominal's action before being sent to the actuators.

The defining property is the *frequency separation*: the nominal policy retains its training-time frequency (e.g. 100 Hz) for stability and learning tractability, while the residual operates at the highest feedback rate that the hardware supports (e.g. 400 Hz on a legged manipulator with a 400 Hz state estimator).

## Intuition

Standard residual policy learning (Silver 2018; Johannink 2019; Luo 2024) trains a residual *at the same rate* as the nominal it corrects. This bakes the nominal's control frequency into the residual's effective bandwidth. For static or low-rate tasks that's fine, but for highly dynamic skills (throwing, hitting, whipping) the bandwidth of the *correction loop* — not the planning loop — is what limits accuracy.

A high-frequency residual asks: how fast can your sensors close a feedback loop, and let *that* be the residual rate. The nominal policy provides the slow open-loop "where to be"; the residual provides the fast closed-loop "fix the tracking error you're accumulating right now." The training reward keeps the residual's action *small*, encouraging it to stay close to the nominal's behavior — it is a high-bandwidth low-magnitude correction, not a re-trained policy.

## Formal notation

Let $\pi_{\text{nom}}: \mathcal{O} \to \mathcal{A}$ be the frozen nominal policy at rate $f_{\text{nom}}$, and let $\pi_{\text{res}}: \mathcal{O} \times \mathcal{C} \to \mathcal{A}$ be the residual at rate $f_{\text{res}} \gg f_{\text{nom}}$, with $\mathcal{C}$ holding extra high-rate context (decimation index $k \in \{0, \dots, K-1\}$ where $K = f_{\text{res}}/f_{\text{nom}}$, plus high-rate targets such as a desired EE acceleration). At every fast-loop step:

$$
a_t = \pi_{\text{nom}}(o_t) + \pi_{\text{res}}(o_t, c_t)
$$

with $a_t$ the actuator command. Train $\pi_{\text{res}}$ with the nominal's reward (on the slow time scale) plus an action-scale penalty $-\lambda \|\pi_{\text{res}}\|^2$ that anchors it to the nominal.

## Variants

- **Acceleration-conditioned residual** (this paper): the residual receives a high-rate desired EE acceleration target as additional input, allowing it to track an externally-supplied dynamic reference that the nominal alone cannot.
- **State-only residual**: residual sees only the nominal's observation (no extra high-rate target). Simpler; gives up acceleration tracking.
- **Differentiable residual into a downstream QP**: the residual's parameters trained to minimize the loss of a downstream model-based optimizer (e.g. tube QP) rather than just the nominal task reward.

## Comparison

- vs. **same-rate residual policy** (Silver 2018; Johannink 2019; Luo 2024): same parametric structure (additive correction on a frozen prior policy), but the residual rate is decoupled from the nominal rate.
- vs. **fine-tuning the nominal policy at a higher rate**: fine-tuning at higher rate destroys the learned slow-time-scale strategy; a *frozen* nominal + small residual preserves it.
- vs. **end-to-end policy at residual rate**: training a single policy at 400 Hz from scratch loses the curriculum structure; the nominal-then-residual decomposition is a deliberate two-stage curriculum.
- vs. **classical inner-loop controllers** (PD, impedance) underneath an RL outer loop: the high-frequency residual is *learned*, so it can absorb model error and unmodeled effects (gear backlash, actuator network mismatch) that a hand-designed inner loop cannot.

## When to use

- The state estimator runs significantly faster than the nominal policy's training rate.
- The task is bandwidth-limited: small, high-rate corrections meaningfully reduce tracking error (high-velocity throwing, whipping, fast pose tracking on dynamic platforms).
- A trained nominal policy is already available and you want to improve it without retraining.
- The nominal action space is continuous and additive corrections are well-defined (joint-position offsets, torque offsets).

Skip when the task is quasi-static, when nominal training already runs at the state-estimator rate, or when the residual would dominate the nominal's behavior (in which case retrain the nominal).

## Known limitations

- **Sim-to-real gap inflation**: a residual trained at 400 Hz can over-fit fine-grained dynamics of the training simulator (gear cogging, actuator delays) more readily than a slower policy. The original paper observes that hardware gains are smaller than simulation gains.
- **Vertical-only / partial residuals**: the original work tracks accelerations only in a single direction. Generalizing to full 3D acceleration tracking is non-trivial.
- **Action-scale penalty tuning**: too small a penalty and the residual diverges from the nominal; too large and it cannot reduce error meaningfully.
- **Two training stages** (nominal then residual) increase total training compute compared to a single policy.

## Open problems

- A residual policy training scheme that closes the sim-to-real gap (training-environment-aware regularizers, real-data fine-tuning, etc.).
- Conditions under which a high-frequency residual provably dominates a same-rate residual.
- Multi-rate residual stacks (nominal → mid-rate residual → fast-rate residual) and whether the diminishing returns curve flattens after two layers.
- Cross-task transfer of a residual: does a residual trained for throwing help with hitting/whipping on the same robot?

## Key papers

- [[learning-accurate-whole-body-throwing-high]] — introduces the 400 Hz residual on top of a 100 Hz nominal for whole-body prehensile throwing on ANYmal-D + DynaArm; ablations confirm monotonic accuracy gains with residual rate up to the state-estimation ceiling.

## My understanding

The high-frequency residual policy is a generalization of TossingBot's residual physics: TossingBot added a *scalar* learned correction to a hand-coded ballistic prior; this paper adds a *continuous high-rate* learned correction to a *learned* nominal policy. The transfer that matters for dynamic deformable manipulation (DeformY): when whipping a flexible cable to a target, the controller has the same high-rate state estimator (visual or proprioceptive cable-tip estimate) and the same gap between a learned nominal motion and a precise terminal state — exactly the regime where a high-frequency residual is the right structural fix.
