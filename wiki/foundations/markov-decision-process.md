---
title: "Markov Decision Process"
slug: "markov-decision-process"
domain: "RL"
status: mainstream
aliases: ["MDP", "Markov decision problem"]
first_introduced: "Bellman 1957"
date_updated: "2026-06-16"
source_url: "https://en.wikipedia.org/wiki/Markov_decision_process"
---

## Definition

A Markov decision process (MDP) is a mathematical model for sequential decision making under uncertainty. It is the standard problem formalism of reinforcement learning, specified by the tuple $(\mathcal{S}, \mathcal{A}, P, R, \gamma)$: a set of states $\mathcal{S}$, a set of actions $\mathcal{A}$, a transition kernel $P(s' \mid s, a)$, a reward function $R(s, a)$ (or $R(s, a, s')$), and a discount factor $\gamma \in [0, 1)$. The defining Markov property is that the next state and reward depend only on the current state and action, not on the full history.

## Intuition

An agent observes the current state, picks an action, receives a reward, and transitions stochastically to a new state; the goal is to choose actions so as to maximize cumulative (discounted) reward over time. "Markov" means the present state is a sufficient statistic of the past: once you know where you are, how you got there does not matter for predicting the future. This compresses an arbitrarily long interaction history into a fixed-size state and makes dynamic programming tractable.

## Formal notation

A policy $\pi(a \mid s)$ maps states to action distributions. The objective is to maximize the expected return $J(\pi) = \mathbb{E}_\pi\!\left[\sum_{t=0}^{\infty} \gamma^t R(s_t, a_t)\right]$. The value functions satisfy the Bellman equations: $V^\pi(s) = \mathbb{E}_{a \sim \pi}\big[R(s,a) + \gamma\,\mathbb{E}_{s'\sim P}[V^\pi(s')]\big]$ and the optimal value obeys $V^*(s) = \max_a \big[R(s,a) + \gamma\,\mathbb{E}_{s'}[V^*(s')]\big]$. The discount $\gamma$ guarantees the infinite-horizon return is finite and trades off short- vs long-term reward.

## Key variants

- **Finite vs infinite horizon** — fixed episode length vs discounted/average-reward over an unbounded horizon.
- **Partially Observable MDP (POMDP)** — the agent sees observations $o$ rather than the true state, requiring belief-state tracking.
- **Continuous MDP** — $\mathcal{S}$ and/or $\mathcal{A}$ are continuous (the typical setting for robot control).
- **Constrained MDP** — adds cost constraints alongside the reward objective.
- **Semi-MDP / options** — actions take variable amounts of time (temporal abstraction).

## Known limitations

Exact solution by dynamic programming scales poorly with state/action dimensionality (the curse of dimensionality). The Markov assumption is often violated in practice (history-dependent or partially observed dynamics), forcing state augmentation or recurrent representations. The transition kernel and reward are usually unknown, which is precisely why model-free and model-based RL exist. Discounting is partly a mathematical convenience and can distort genuinely undiscounted long-horizon objectives.

## Open problems (LLM analysis)

Scalable planning in high-dimensional continuous MDPs (e.g. deformable-object manipulation); principled handling of partial observability without intractable belief updates; reward specification that faithfully encodes designer intent; and bridging the gap between the clean MDP abstraction and messy real-world non-stationarity.

## Relevance to active research (LLM analysis)

The MDP is the substrate beneath every RL paper, including the meta-learned adaptation and model-based planning methods (Learning-to-Adapt, RMA, PETS) being ingested here: each treats control as an MDP and differs only in how it estimates dynamics, adapts to a shifting transition kernel, or plans over the model.
