---
title: "Reward Shaping"
slug: "reward-shaping"
domain: "RL"
status: mainstream
aliases: ["potential-based reward shaping", "PBRS", "dense reward design"]
first_introduced: "Ng, Harada & Russell 1999 (potential-based shaping)"
date_updated: "2026-06-16"
source_url: ""
---

## Definition (LLM analysis)

Reward shaping is the practice of adding an auxiliary reward term $F(s, a, s')$ to a task's (often sparse) reward $R$ so as to provide denser, more frequent learning signal and accelerate reinforcement learning. The central theoretical result is that *potential-based* reward shaping — where $F(s, a, s') = \gamma\,\Phi(s') - \Phi(s)$ for some state potential $\Phi$ — leaves the set of optimal policies unchanged, guaranteeing the agent solves the original task and is not merely chasing the shaping bonus.

## Intuition (LLM analysis)

Sparse rewards (e.g. +1 only on task success) give the agent almost no gradient to climb: it must stumble onto success by chance before it can learn anything. Shaping supplies intermediate "you're getting warmer" signal — distance-to-goal, progress, or sub-goal completion. Potential-based shaping is the safe way to do this: because the bonus is a telescoping difference of a state potential, its contribution to any complete trajectory's return is fixed by the endpoints, so it cannot create reward "loops" that the agent could exploit instead of finishing the task.

## Formal notation (LLM analysis)

Define the shaped reward $R'(s,a,s') = R(s,a,s') + F(s,a,s')$. Ng et al. (1999) proved that if and only if $F$ is potential-based, i.e.
$$F(s,a,s') = \gamma\,\Phi(s') - \Phi(s),$$
then every optimal policy of the shaped MDP is optimal in the original MDP and vice versa. Equivalently, shaping with potential $\Phi$ is equivalent to initializing the value function with $\Phi$. Extensions cover potential-based *advice* $F = \gamma\,\Phi(s',a') - \Phi(s,a)$ and time-varying potentials.

## Key variants (LLM analysis)

- **Potential-based reward shaping (PBRS)** — the policy-invariant, theoretically grounded form.
- **Potential-based advice** — potentials over state-action pairs to inject action preferences.
- **Dynamic / learned potentials** — $\Phi$ that changes over training or is learned online.
- **Heuristic dense rewards** — hand-crafted progress terms that are *not* potential-based (effective but can change the optimum).
- **Curriculum and intrinsic motivation** — related signal-densification via task ordering or novelty bonuses.

## Known limitations (LLM analysis)

Non-potential-based shaping can silently change the optimal policy, producing reward hacking or degenerate behaviors. Designing a good potential $\Phi$ effectively requires partial knowledge of the value function — often as hard as the original problem. Poorly scaled shaping can dominate the true reward, and shaping that helps one task may not transfer.

## Open problems (LLM analysis)

Automatically learning effective, policy-invariant potentials; scaling potential-based shaping to high-dimensional continuous control; detecting and preventing reward hacking from non-potential heuristics; and unifying shaping with intrinsic motivation under one principled framework.

## Relevance to active research (LLM analysis)

Reward shaping is the workhorse that makes RL tractable on real robot tasks with otherwise sparse success signals; adaptation and model-based controllers (RMA, PETS, Learning-to-Adapt) are typically trained against shaped or dense cost/reward functions, so understanding policy-invariance is essential to reading their training setups correctly.
