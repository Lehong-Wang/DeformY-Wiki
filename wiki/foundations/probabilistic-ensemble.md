---
title: "Probabilistic Ensemble"
slug: "probabilistic-ensemble"
domain: "general"
status: mainstream
aliases: ["probabilistic ensembles", "PE", "deep ensemble", "PETS dynamics model"]
first_introduced: "Lakshminarayanan et al. 2017 (deep ensembles); Chua et al. 2018 (PETS)"
date_updated: "2026-06-16"
source_url: ""
---

## Definition (LLM analysis)

A probabilistic ensemble is a collection of probabilistic neural-network models, each of which outputs a parameterized predictive distribution (typically a Gaussian mean and variance) rather than a point estimate. Used as a dynamics model in model-based reinforcement learning, the ensemble jointly captures two distinct sources of uncertainty: aleatoric uncertainty (irreducible stochasticity in the system, modeled by each network's predicted output variance) and epistemic uncertainty (model uncertainty from limited data, exposed by disagreement among ensemble members). It is the dynamics-model backbone of PETS and the principal defense against model exploitation.

## Intuition (LLM analysis)

A single deterministic model, when used for planning, can be "gamed": the planner discovers fictitious high-reward trajectories in regions where the model is confidently wrong. A probabilistic ensemble blunts this in two ways. Each member admits it is unsure (wide predicted variance) where the data is noisy, and the members disagree (spread of predictions) where data is scarce. Propagating this combined uncertainty through planning makes the planner appropriately conservative in exactly the regions where the model cannot be trusted, so imagined returns stay grounded.

## Formal notation (LLM analysis)

The ensemble is $\{\hat{P}_{\phi_b}\}_{b=1}^{B}$, where each member predicts a Gaussian over the next state:
$$\hat{P}_{\phi_b}(s_{t+1} \mid s_t, a_t) = \mathcal{N}\!\big(\mu_{\phi_b}(s_t,a_t),\ \Sigma_{\phi_b}(s_t,a_t)\big).$$
Each member is trained by maximum likelihood (Gaussian NLL), $\mathcal{L}(\phi_b) = \tfrac{1}{2}(\Delta s - \mu_{\phi_b})^\top \Sigma_{\phi_b}^{-1}(\Delta s - \mu_{\phi_b}) + \tfrac{1}{2}\log\det\Sigma_{\phi_b}$, on a bootstrapped data subset. Per-member variance $\Sigma_{\phi_b}$ encodes **aleatoric** uncertainty; the spread $\mathrm{Var}_b[\mu_{\phi_b}]$ across members encodes **epistemic** uncertainty. PETS propagates state distributions by sampling particles and assigning each to a randomly chosen member (the TS$\infty$ / TS1 trajectory-sampling schemes).

## Key variants (LLM analysis)

- **Deterministic ensemble** — multiple point-estimate models; captures epistemic disagreement only.
- **Probabilistic (single) network** — predicts mean+variance; captures aleatoric only, no epistemic spread.
- **Probabilistic ensemble (PE)** — both, as used in PETS; the standard MBRL dynamics model.
- **Bootstrapped vs shared-data ensembles** — bootstrap resampling vs only differing random init/seed.
- **Trajectory sampling schemes (TS1, TS∞)** — how particles are bound to members during multi-step rollouts.

## Known limitations (LLM analysis)

Training and rolling out $B$ networks multiplies compute and memory. Ensemble disagreement is a heuristic, not a calibrated posterior, and can be miscalibrated far from data. Small ensembles ($B \approx 5$) may underestimate epistemic uncertainty. Gaussian predictive heads cannot represent multimodal transitions (e.g. contact making/breaking), a real concern for deformable and contact-rich dynamics.

## Open problems (LLM analysis)

Cheap, well-calibrated epistemic uncertainty without large ensembles; expressive (non-Gaussian, multimodal) probabilistic dynamics for contact-rich and deformable systems; principled uncertainty propagation over long horizons; and tighter coupling between the uncertainty estimate and the planner's risk sensitivity.

## Relevance to active research (LLM analysis)

The probabilistic ensemble is the literal model class inside PETS and is reused throughout sample-efficient model-based control and uncertainty-aware planning; isolating it as a foundation lets the ingested PETS and Learning-to-Adapt pages wikilink to a shared definition instead of duplicating the aleatoric/epistemic distinction.
