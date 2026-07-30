---
name: "Chelsea Finn"
affiliation: "Stanford University; Google DeepMind (at time of this work: UC Berkeley)"
research_areas: [meta-learning, robot-learning, reinforcement-learning, imitation-learning, deep-learning]
homepage: "https://ai.stanford.edu/~cbfinn/"
scholar: "https://scholar.google.com/citations?user=vfPE6hgAAAAJ"
date_updated: 2026-06-16
type:
  kind: researcher
---

## Research areas

- [[model-based-planning-for-manipulation]]
Meta-learning, robot learning, and reinforcement/imitation learning. Originator of Model-Agnostic Meta-Learning (MAML), which underpins the gradient-based fast-adaptation machinery used across robot-learning systems — including the online meta-learned dynamics adaptation of [[learning-adapt-dynamic-real-world-environments]] (GrBAL builds directly on MAML). Recurring senior author on the "learn to adapt fast" line that connects meta-learning to real-world robot control.

## Recent work

- [[learning-adapt-dynamic-real-world-environments]] — model-based meta-RL for fast online dynamics adaptation (GrBAL/ReBAL); first meta-RL deployment on a real robot.

## My notes

Finn is the connective node between the meta-learning foundation ([[meta-learning]]) and its application to online robot adaptation. For the rope/DLO arc of this wiki, her MAML-and-fast-adaptation lineage is the meta-RL parent of the more specialized online-adaptation mechanisms (delta-dynamics, implicit sysID, RMA-style distillation) that the DLO papers use.
