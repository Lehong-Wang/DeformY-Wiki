---
title: "Cross-Entropy Method"
slug: "cross-entropy-method"
domain: "general"
status: mainstream
aliases: ["CEM", "cross entropy method"]
first_introduced: "Rubinstein 1997"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Cross-entropy_method"
---

## Definition

The cross-entropy (CE) method is a Monte Carlo, derivative-free approach to importance sampling and optimization. It is applicable to both combinatorial and continuous problems with static or noisy objectives. For optimization it works by maintaining a parameterized sampling distribution over candidate solutions and iteratively refitting that distribution toward the highest-scoring ("elite") samples, so that probability mass concentrates around the optimum.

## Intuition

CEM turns optimization into iterative distribution-fitting. Sample a batch of candidates from the current distribution, evaluate them, keep the top fraction (the elites), and re-estimate the distribution's parameters from just those elites. Repeat. The sampling distribution marches toward the region of best solutions without ever needing a gradient — only the ability to evaluate (sample-and-score) the objective. This makes it ideal for black-box costs such as rolling out a learned dynamics model and reading off the return.

## Formal notation

To maximize $S(x)$, maintain a sampling distribution $p(\cdot;\, \theta)$ (commonly a diagonal Gaussian $\mathcal{N}(\mu, \mathrm{diag}(\sigma^2))$ over action sequences). Each iteration: draw $x_1,\dots,x_N \sim p(\cdot;\theta)$, compute scores $S(x_i)$, select the elite set $E$ of the top $\rho N$ samples, and update by maximum likelihood on the elites:
$$\mu \leftarrow \frac{1}{|E|}\sum_{i\in E} x_i, \qquad \sigma^2 \leftarrow \frac{1}{|E|}\sum_{i\in E}(x_i - \mu)^2.$$
The name comes from this fit minimizing the cross-entropy (KL divergence) between the updated distribution and the elite-induced distribution.

## Key variants

- **CEM for optimization vs rare-event estimation** — the same machinery solves both maximization and importance-sampling of rare events.
- **Gaussian CEM (continuous control)** — diagonal-Gaussian sampling over action sequences; the standard sampling-based MPC planner.
- **CEM with momentum / colored noise** — smoother updates and temporally correlated action noise for better exploration.
- **CMA-ES** — a closely related evolutionary method that additionally adapts a full covariance matrix and step size.
- **MPPI** — a reward-weighted (softmax) sampling-based planner often used interchangeably with CEM in MBRL.

## Known limitations

Cost scales with the number of samples times rollout length, which is expensive over long horizons or large action dimensions. Diagonal-Gaussian sampling can collapse prematurely or get stuck in local optima. Performance is sensitive to population size, elite fraction, and the number of iterations, and the planner inherits any inaccuracies of the learned model it rolls out.

## Open problems (LLM analysis)

Sample-efficient sampling-based planning for high-dimensional and long-horizon control; avoiding premature variance collapse; learned or amortized proposal distributions to warm-start CEM; and tighter integration with model uncertainty so the planner does not exploit model error.

## Relevance to active research (LLM analysis)

CEM is the default population-based planner inside model-based RL and sampling MPC: PETS plans action sequences with CEM over its probabilistic-ensemble model, and many MBRL controllers (including adaptation-time replanning) reuse it. Seeding it as a foundation lets the ingested PETS/Learning-to-Adapt pages link to a single shared definition of the planner.
