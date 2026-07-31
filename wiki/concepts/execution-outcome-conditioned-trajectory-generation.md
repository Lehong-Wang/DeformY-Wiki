---
title: "Execution-Outcome-Conditioned Trajectory Generation"
aliases: ["outcome-conditioned trajectory generation", "achieved-outcome conditioning", "executed-landing conditioning", "trajectory-level dynamics learning", "hindsight-outcome conditioning", "conditioning on what happened"]
tags: [goal-conditioned, generative-model, hindsight-relabeling, dynamics-gap, sim-to-real, inverse-model, trajectory-generation, dynamic-manipulation, data-efficiency, robot-learning]
maturity: emerging
definition: "Training a goal-conditioned trajectory generator by labeling each executed trajectory with the outcome it actually achieved rather than the outcome it was commanded to achieve, so the model learns the executed trajectory-to-outcome map directly and absorbs the dynamics gap implicitly instead of modeling or correcting it."
key_papers:
- "[[da-mmp-learning-coordinated-accurate-throwing]]"
first_introduced: "2025"
date_updated: 2026-07-30
related_concepts:
- "[[residual-physics]]"
- "[[iterative-residual-policy]]"
- "[[motion-manifold-primitives]]"
parent_topic: dynamic-throwing-and-hitting
---

## Definition

Let $\pi_\theta(\tau \mid c)$ be a generative model over trajectories (or trajectory parameters) conditioned on a goal $c$. **Execution-outcome-conditioned training** builds the dataset by executing trajectories $\tau_i$, *measuring* the outcome $o_i$ each one actually produced, and training on pairs $(\tau_i, c{=}o_i)$ — never on the target $c_i^{\mathrm{cmd}}$ that $\tau_i$ was generated for. At inference the desired target is fed into the same conditioning slot: $\tau \sim \pi_\theta(\cdot \mid c = g^\star)$.

The model therefore learns the **executed** trajectory→outcome relation, inverted. Nothing anywhere in the pipeline represents "the gap between planned and executed", because the gap has no place to appear: every training label was already measured downstream of it.

Two properties follow immediately, and they are the reason to name this as its own concept:

1. **Every rollout is a valid training datum.** A trajectory that misses its commanded target by 30 cm is not a failure; it is an exact-hit demonstration for wherever it did land. Success rate stops gating data yield.
2. **The correction is indexed by the trajectory, not by the goal.** Two distinct trajectories aimed at the same target that land in different places are two consistent data points, not a contradiction — which is not true of any target-indexed residual.

## Intuition

The standard pipeline is: analytic/planned model produces a nominal action for a target, reality deviates, learn a correction. Every stage of that chain has to *represent* the deviation — as a residual on the action ([[residual-physics]]), as delta-dynamics ([[delta-dynamics-network]]), as an identified parameter ([[implicit-system-identification]]).

Outcome conditioning deletes the chain. If you only ever train on what really happened, the learned inverse map is already the real one. The nominal model's job shrinks to *proposing diverse feasible motion* — its accuracy stops mattering entirely.

This is the hindsight-relabeling principle (HER / GCSL) transplanted from per-step RL into whole-trajectory generative modeling. The decisive practical consequence is data cost: the two published instances need **60 real trials** (DA-MMP) and can tolerate a systematic implementation bug corrupting every one of them, because the labels sit downstream of the corruption.

## Variants

The axis that actually separates instances is **how many outcome labels one executed rollout yields**.

- **One label per rollout (trajectory-level).** [[da-mmp-learning-coordinated-accurate-throwing]]: each executed ring toss is labeled with its single measured landing point $(x_{\mathrm{exe}}, y_{\mathrm{exe}})$ at the peg height; 60 executions ⇒ 60 pairs. Forced by the task — a released projectile has exactly one meaningful outcome.
- **Many labels per rollout (per-timestep).** [[per-timestep-hindsight-relabeling]]: when the object of interest is *continuously observable in goal space* (a rope tip sweeping through positions and velocities), every timestep of one rollout is an exact hit on the state it passed through. A 3 s rollout at 60 Hz ⇒ ~10² pairs at zero extra cost. This is only available when goals are *passed through* rather than *terminal*.
- **Conditioning richness.** DA-MMP conditions on a 2-D landing position; the rope-swing project conditions on 5-D (position + arrival direction). Richer outcome vectors reduce inverse multimodality — different arrival directions separate the swing families that share a position — at the cost of a larger conditioning space to cover.
- **Where the generator lives.** Over an autoencoded latent manifold (DA-MMP, on top of [[motion-manifold-primitives]]) or directly over primitive parameters ([[conditional-flow-matching-motion-parameters]]).

## Comparison

- vs **[[residual-physics]]** (TossingBot, whole-body throwing): residuals cancel a *bias* on top of an analytic prior. DA-MMP's diagnostic is the sharp version — the same residual-style correction scores 93.3% in simulation, where the gap is nearly pure bias, and **6.7%** in the real world, where the gap has large variance; worse than simply re-throwing (23.3% vs 13.3% for one/two blind attempts). Outcome conditioning does not distinguish bias from variance because it never separates nominal from correction.
- vs **[[iterative-residual-policy]] / [[delta-dynamics-network]]**: those spend *per-goal real trials at deployment* to iterate onto the target. Outcome conditioning spends its trials once, offline, and deploys zero-shot — the trade is deployment cost against the requirement that the outcome distribution be roughly stationary.
- vs **[[implicit-system-identification]] / [[optimization-based-inverse-model]]**: those identify or invert a *model*. Outcome conditioning never builds one; the generative model is the inverse, amortized.
- vs **plain goal-conditioned imitation on planned data**: identical architecture, different labels — and that difference is the whole method. Training the same model on commanded targets reproduces the planner's errors exactly.

## Known limitations

- **Stationarity assumption.** The learned inverse is valid for the hardware, object, and grasp that produced the trials. A new rope, gripper, or payload invalidates the outcome labels (though not the underlying motion manifold).
- **Coverage becomes the binding resource.** With exploration no longer required, the failure mode moves to reachability: a target outside the executed outcome distribution has no support. DA-MMP generalizes from $[1.5, 2.0]$ m training to 1.2 m, but the extent of safe extrapolation is uncharacterized.
- **Requires measurable outcomes.** The outcome must be observable at deployment-time precision — a vision pipeline error enters directly as label noise. DA-MMP's real-world variance is partly its RealSense/ellipse-fit localization.
- **Silent about multimodality.** Conditioning on the achieved outcome makes the map one-to-many by construction (many trajectories land in one place), so a deterministic regressor will average across incompatible families; the generator must be genuinely distributional.
- **No guarantee on any single sample.** Because the model is distributional, one draw can be an outlier. DA-MMP executes exactly one sample and does nothing about this; [[sim-verified-best-of-n-selection]] is the obvious complement.

## Open problems

- Does the per-timestep variant actually deliver its ~10² data multiplier in learned accuracy, or does label correlation within a rollout eat most of it? Untested — the rope-swing project's Stage A/B is the test.
- How rich can the conditioning vector get before coverage becomes infeasible? 2-D is demonstrated; 5-D (position + direction) is the open case.
- Does the trick survive *state-dependent* dynamics gaps — where the discrepancy varies with the trajectory's own speed or shape — rather than a roughly stationary one?
- How to detect at inference time that a requested goal is outside the executed-outcome support, before executing an open-loop motion.
- Can outcome labels be transferred or fine-tuned across hardware, so that a new rope needs 5 trials instead of 60?

## Relationship to foundations

An application of [[imitation-learning]]'s supervised-inverse-map machinery where the labels are self-generated by execution rather than by a demonstrator, with the relabeling logic borrowed from hindsight experience replay in the [[markov-decision-process]] setting but applied at whole-trajectory granularity. It is a *substitute* for explicit [[system-identification]] and for [[sim-to-real-transfer]] by domain randomization: rather than closing the gap, it retrains the inverse map on the far side of it.

## Realized by

- [[da-mmp-dynamics-aware-motion-manifold]] — DA-MMP: latent-space conditional flow matching trained on 60 executed throws labeled by measured landing point.
- [[per-timestep-hindsight-relabeling]] — the per-timestep variant, emitting one exact-hit pair per rollout timestep.

## My understanding

This is the concept the rope-swing project's data engine actually instantiates, and naming it separates two claims that are easy to conflate. The *principle* — train on achieved outcomes, not commanded ones — is not ours; [[da-mmp-learning-coordinated-accurate-throwing]] published it for throwing and demonstrated its payoff, including the accidental natural experiment where a systematic implementation bug corrupted every executed trajectory and the method absorbed it. What is ours is the **label multiplicity**: DA-MMP gets one label per rollout because a released ring lands once, while a rope tip is continuously in goal space and yields ~10² exact-hit labels per rollout. That distinction is defensible on task structure rather than on cleverness, which makes it a better novelty claim than "we thought of relabeling".

The transferable warning is in the limitations: outcome labels are hardware-and-object-specific. Our 90k-equivalent swing manifold survives a rope change; our outcome-conditioned amortizer does not. That is the argument for the project's per-rope calibration budget being spent on relabeling data, not on system identification.
