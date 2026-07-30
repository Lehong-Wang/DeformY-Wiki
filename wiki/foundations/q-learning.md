---
title: "Q-Learning"
slug: "q-learning"
domain: "RL"
status: mainstream
aliases: ["Q learning", "deep Q-network", "DQN"]
first_introduced: "Watkins 1989"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Q-learning"
---

## Definition

Q-learning is a model-free, off-policy reinforcement learning algorithm that learns the optimal action-value function $Q^*(s, a)$ — the expected discounted return of taking action $a$ in state $s$ and acting optimally thereafter — without requiring a model of the environment's transitions or rewards. Once $Q^*$ is known, the optimal policy is greedy: $\pi^*(s) = \arg\max_a Q^*(s, a)$.

## Intuition

The agent maintains an estimate of "how good" each action is in each state and improves it by bootstrapping: after taking an action and observing the reward and next state, it nudges its current estimate toward the immediate reward plus its own best guess of future value. Because the update targets the *maximum* over next-state actions regardless of the action actually taken, learning is off-policy — the agent can learn the optimal policy while exploring with a different (e.g. $\epsilon$-greedy) behavior policy.

## Formal notation

The tabular Q-learning update for a transition $(s, a, r, s')$ is
$$Q(s,a) \leftarrow Q(s,a) + \alpha\Big[\, r + \gamma \max_{a'} Q(s', a') - Q(s,a) \,\Big],$$
where $\alpha$ is the learning rate and the bracketed term is the temporal-difference error. Under standard stochastic-approximation conditions (all state-actions visited infinitely often, suitable step-size decay), tabular Q-learning converges to $Q^*$. Deep Q-learning replaces the table with a function approximator $Q_\theta$ trained to minimize the squared TD error against a slowly-updated target network.

## Key variants

- **Tabular Q-learning** — the original lookup-table algorithm with convergence guarantees.
- **Deep Q-Network (DQN)** — neural function approximation with experience replay and a target network.
- **Double Q-learning / Double DQN** — decouples action selection from evaluation to reduce maximization bias.
- **Dueling DQN** — separates state-value and advantage streams.
- **Distributional / Rainbow** — models the full return distribution and combines multiple DQN improvements.

## Known limitations

The $\max$ operator induces overestimation bias (addressed by Double Q-learning). It applies most naturally to discrete action spaces; continuous actions require an inner maximization or actor-critic reformulation (e.g. DDPG). Function approximation, bootstrapping, and off-policy data together form the "deadly triad" that can cause divergence. Sample efficiency and stability on real systems remain challenging.

## Open problems (LLM analysis)

Stable value learning under the deadly triad; sample-efficient Q-learning for continuous high-dimensional control; bias-corrected target estimation beyond double estimators; and integrating value functions with learned dynamics models for planning.

## Relevance to active research (LLM analysis)

Q-learning is the canonical value-based counterpart to policy-gradient methods and underlies many actor-critic controllers (SAC, TD3) used as the policy backbone in adaptation and model-based pipelines; it is included here as core RL background so that ingested method papers can wikilink to it instead of re-deriving value learning.
