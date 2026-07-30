---
title: "Trajectory Sampling Uncertainty Propagation"
aliases: ["trajectory sampling", "TS propagation", "TS1", "TS-infinity", "particle-based uncertainty propagation", "ensemble particle propagation"]
tags: [model-based-reinforcement-learning, probabilistic-ensemble, uncertainty-propagation, aleatoric-uncertainty, epistemic-uncertainty, planning, particle-method]
maturity: stable
definition: "A particle-based scheme that propagates state particles through a probabilistic ensemble dynamics model — each particle bound to a bootstrap — to represent (and, in the TS-infinity variant, separate) aleatoric vs. epistemic uncertainty over a planning horizon."
key_papers: ["[[deep-reinforcement-learning-handful-trials-using]]"]
first_introduced: "2018"
date_updated: 2026-06-16
related_concepts: ["[[planning-as-diffusion]]"]
parent_topic: "[[model-based-planning-for-manipulation]]"
---

## Definition

**Trajectory sampling (TS)** is the uncertainty-propagation method introduced by PETS for planning with a probabilistic ensemble dynamics model. Given a candidate action sequence and a learned ensemble $\{\tilde f_{\theta_b}\}_{b=1}^B$ of probabilistic networks, TS creates $P$ state particles initialized at the current state ($s^p_{t=0}=s_0\ \forall p$) and steps each particle by

$$
s^p_{t+1} \sim \tilde f_{\theta_{b(p,t)}}\!\big(s^p_t, a_t\big),
$$

where $b(p,t)\in\{1,\dots,B\}$ is the bootstrap index assigned to particle $p$ at time $t$, and the sample is drawn from that network's predicted (Gaussian) output distribution. The resulting particle cloud is a Monte-Carlo approximation of the model-induced trajectory distribution, used to score actions by mean per-particle reward inside MPC. The defining choice is how $b(p,t)$ evolves, which controls how the two uncertainty types mix.

## Intuition

A learned dynamics model is uncertain for two different reasons, and a planner must not confuse them. **Aleatoric** uncertainty is irreducible system noise; **epistemic** uncertainty is "I have not seen enough data here," which shrinks with experience and is the only uncertainty worth *exploring*. Collapsing the model to a single mean prediction (the Expectation baseline) throws both away and lets small biases compound over the horizon; moment-matching (MM) or distribution-sampling (DS) recast the state to a Gaussian each step and entangle the two uncertainties. TS instead keeps a *population* of particles — naturally representing multimodal trajectory distributions — and, by binding particles to bootstraps, can keep epistemic and aleatoric variance bookkept separately.

## Variants

- **TS1** — each particle re-samples its bootstrap index $b(p,t)$ *uniformly every timestep*. This corresponds to continually re-sampling from the ensemble's approximate marginal posterior of plausible dynamics, and places a soft cap on trajectory multimodality: particle separation cannot accumulate from a single particle persistently following one (possibly outlier) bootstrap.
- **TS∞** — each particle keeps its bootstrap index *fixed for the entire trial*, respecting the (assumed) time-invariance of the true dynamics. This is the variant that makes uncertainties **separable**: aleatoric state variance = average variance of particles sharing a bootstrap; epistemic state variance = variance of the per-bootstrap particle means. Only the epistemic component is the "learnable" uncertainty useful for directed exploration.

## Comparison

Against the baseline propagation methods PETS ablates:

- **Expectation (E):** single mean particle, no uncertainty — cheapest, but model bias compounds with no signal of estimate quality.
- **Moment Matching (MM):** recast the state distribution to a single Gaussian each step — competitive in low dimensions (consistent with Deep-PILCO) but unreliable in high dimensions (half-cheetah); forces unimodality.
- **Distribution Sampling (DS):** moment-match over bootstraps only — an intermediate softening of multimodality between MM and TS.
- **TS (this concept):** particle population, multimodal-capable, and (TS∞) uncertainty-separating. Empirically, *given a good probabilistic-ensemble model*, the propagation choice matters only modestly — the dominant lever is uncertainty at model-*learning* time — but TS is what lets the planner exploit the ensemble's full predictive distribution and is required for any downstream epistemic-driven exploration.

## Known limitations

- Separating epistemic from aleatoric uncertainty (the TS∞ payoff) is demonstrated, but the original work does **not** yet use the epistemic signal to drive exploration.
- TS∞'s separability relies on **time-invariant** dynamics; it does not, by itself, handle dynamics that drift within a trial.
- Particle propagation cost scales with $P$ (and the ensemble size $B$), adding compute over a single deterministic rollout.
- Calibration of the separated uncertainties is only as good as the bootstrapped-ensemble proxy for the Bayesian posterior.

## Open problems

- Turning the separated **epistemic** state variance into a principled, calibrated exploration bonus for MBRL.
- Whether TS-style particle propagation over an ensemble remains the right propagation choice for **high-dimensional deformable-object** dynamics (rope/cloth), where short-horizon uncertainty-aware planning is most needed and a single global model is hopeless.
- Selecting between TS1 and TS∞ (and $P$) in a task-aware way rather than by fixed default.

## Relationship to foundations

- [[probabilistic-ensemble]] — TS is the propagation method *for* a probabilistic-ensemble dynamics model; the bootstrap structure of the PE is exactly what TS exploits to separate uncertainties.
- [[model-predictive-control]] — TS supplies the trajectory distribution that MPC scores when optimizing each action sequence.
- [[model-based-reinforcement-learning]] — TS is the uncertainty-propagation stage of the PETS MBRL loop.

## Realized by

- [[pets-probabilistic-ensembles-trajectory-sampling]] — the PETS algorithm, which uses TS (TS1 / TS∞) to propagate its probabilistic-ensemble model under CEM-MPC.

## My understanding

The quiet importance of TS is that it is the bridge between "I have an *ensemble of probabilistic* models" and "my *planner* can act on that uncertainty." An ensemble that captures both uncertainty types is useless to a controller that collapses it; TS∞ is the bookkeeping trick that keeps the two apart all the way to the action score, which is what makes uncertainty-aware planning (and, eventually, epistemic-driven exploration) possible. For learned-model rope/DLO planning, this is the conceptual hook for *not* exploiting the model where it is least trustworthy — the per-bootstrap disagreement of the particle cloud is precisely a map of where the planner should distrust its own model.
