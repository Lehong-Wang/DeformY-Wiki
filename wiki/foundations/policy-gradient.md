---
title: "Policy Gradient Methods"
slug: "policy-gradient"
domain: "RL"
status: mainstream
aliases: ["policy gradient", "REINFORCE", "actor-critic"]
first_introduced: "Williams 1992 (REINFORCE); Sutton et al. 2000 (policy gradient theorem)"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Policy_gradient_method"
---

## Definition

Policy gradient methods are a class of reinforcement learning algorithms that directly optimize a parameterized, differentiable policy $\pi_\theta(a \mid s)$ by ascending the gradient of expected return, rather than first learning a value function and deriving a policy from it. They are the canonical approach when the action space is continuous or the optimal policy is stochastic.

## Intuition

Instead of asking "how good is each action?" (value-based) and acting greedily, policy gradient asks "how should I nudge my policy parameters so that trajectories that earned high reward become more likely?" Actions that led to above-average returns are reinforced; those that led to below-average returns are suppressed. Because the policy is updated smoothly via gradients, the method extends naturally to high-dimensional continuous controls like robot torques.

## Formal notation

The policy gradient theorem gives an unbiased gradient of the return $J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}[R(\tau)]$:
$$\nabla_\theta J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}\!\left[\sum_t \nabla_\theta \log \pi_\theta(a_t \mid s_t)\, \Psi_t\right],$$
where $\Psi_t$ is a credit-assignment signal — the Monte Carlo return (REINFORCE), the action-value $Q^\pi$, or, with lowest variance, the advantage $A^\pi(s_t,a_t) = Q^\pi - V^\pi$. A learned baseline/critic $V_\phi$ reduces variance without adding bias, yielding actor-critic methods.

## Key variants

- **REINFORCE** — vanilla Monte Carlo policy gradient with an optional baseline.
- **Actor-critic** — a learned value critic supplies the advantage estimate (e.g. A2C/A3C, GAE).
- **TRPO** — constrains each update to a trust region (bounded KL step) for monotonic improvement.
- **PPO** — a clipped surrogate objective approximating the trust region; the de-facto default for continuous control.
- **DDPG/TD3/SAC** — deterministic or maximum-entropy policy gradients for off-policy continuous control.

## Known limitations

High gradient variance and sample inefficiency, especially with sparse rewards. Sensitivity to step size — too large a step can collapse the policy (the motivation for TRPO/PPO). On-policy variants discard data after each update. Convergence is typically only to a local optimum, and reward scale/normalization strongly affects stability.

## Open problems (LLM analysis)

Sample-efficient on-policy learning on real robots; stable optimization under non-stationary dynamics; variance reduction beyond baselines; and integrating policy gradients with learned models so that imagined rollouts can supply low-variance gradient signal.

## Relevance to active research (LLM analysis)

Policy gradient is the optimizer of choice for the learned controllers in RMA-style and model-based pipelines: meta-RL and adaptation methods often train the base policy with PPO/SAC, then layer fast adaptation (context inference or gradient steps) on top, making this foundation a prerequisite for the method papers being ingested.
