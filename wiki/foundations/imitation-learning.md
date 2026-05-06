---
title: "Imitation Learning"
slug: "imitation-learning"
domain: "Robotics"
status: mainstream
aliases: ["learning from demonstration", "apprenticeship learning"]
first_introduced: ""
date_updated: "2026-05-06"
source_url: "https://en.wikipedia.org/wiki/Imitation_learning"
---

## Definition

Imitation learning is a paradigm in reinforcement learning, where an agent learns to perform a task by supervised learning from expert demonstrations. It is also called learning from demonstration and apprenticeship learning.

## Intuition (LLM analysis)

Specifying reward functions for real-world manipulation is hard; collecting expert demonstrations is often easier. Imitation turns the policy-search problem into a supervised-learning problem on (state, expert-action) pairs.

## Formal notation (LLM analysis)

Given demonstrations $\mathcal{D} = \{(s_i, a_i)\}$ from expert policy $\pi^*$, learn $\pi_\theta$ that minimizes $\mathbb{E}_{s\sim d^{\pi_\theta}} \ell(\pi_\theta(s), \pi^*(s))$.

## Key variants

- Behavioral cloning (direct supervised regression).
- DAgger (interactive expert relabeling to fix compounding error).
- Inverse reinforcement learning (recover the reward, then plan).
- GAIL / AIRL (adversarial matching of state-action distributions).
- Diffusion policy (multi-modal action modeling).

## Known limitations (LLM analysis)

Suffers from compounding distribution shift when the policy strays from demonstrated states. Cannot exceed expert performance without additional signal. Sensitive to demonstration quality and embodiment mismatch.

## Open problems (LLM analysis)

Cross-embodiment transfer; safely extrapolating beyond demonstrated states; combining imitation with reward feedback (RLHF-style for robotics); long-horizon goal-conditioned imitation.

## Relevance to active research (LLM analysis)

Imitation learning is the dominant paradigm behind modern robot manipulation foundation models (RT-X, OpenVLA, π0) and DLO manipulation policies trained from teleoperated demonstrations.
