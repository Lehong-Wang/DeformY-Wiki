---
title: "Model-Based Reinforcement Learning"
slug: "model-based-reinforcement-learning"
domain: "Robotics"
status: mainstream
aliases: ["MBRL", "world-model RL"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: ""
---

## Definition

Model-based RL is the family of RL algorithms that learn (or are given) a forward dynamics model and use it to plan, simulate rollouts, or compute analytic policy gradients, rather than only learning a policy or value function from environment interaction.

## Intuition (LLM analysis)

If you have a good model, you can reason about consequences without actually acting; this is far cheaper than collecting real-robot rollouts. Model errors compound, so MBRL pairs the model with uncertainty handling (ensembles, Bayesian models) and short planning horizons.

## Formal notation (LLM analysis)

Learn $\hat P_\phi(s_{t+1} \mid s_t, a_t)$. Optimize policy via planning ($a_t = \arg\max \sum \hat r$), model rollouts (Dyna-style), or differentiable simulation $(\nabla_\theta J = \nabla_\theta \sum \hat r)$.

## Key variants (LLM analysis)

- PILCO (Gaussian-process dynamics + analytic gradient).
- PETS (probabilistic ensemble + CEM/MPC).
- Dreamer / DreamerV3 (latent world models, actor-critic in imagination).
- MuZero (planning over learned latent dynamics + policy/value).
- Differentiable simulation (BPTT through analytic / learned physics).

## Known limitations (LLM analysis)

Compounding model error. Difficult on long horizons. Requires good uncertainty estimates to avoid exploiting model bias. Latent world models are hard to interpret and debug.

## Open problems (LLM analysis)

World models for deformable / contact-rich physics; model-based exploration with calibrated uncertainty; combining learned and analytical dynamics.

## Relevance to active research (LLM analysis)

DLO manipulation begs for model-based methods (rollouts are expensive, demonstrations are scarce), and learned latent dynamics are a natural fit for the high-dimensional rope state.
