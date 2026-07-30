---
name: "Sergey Levine"
affiliation: "UC Berkeley (EECS); Google DeepMind"
research_areas: [reinforcement-learning, robot-learning, model-based-reinforcement-learning, deep-learning, imitation-learning, offline-rl]
homepage: "https://people.eecs.berkeley.edu/~svlevine/"
scholar: "https://scholar.google.com/citations?user=8R35rCwAAAAJ"
date_updated: 2026-06-16
type:
  kind: researcher
---

## Research areas

- [[sim-to-real-and-rapid-adaptation]]
- [[model-based-planning-for-manipulation]]
Reinforcement learning and robot learning, with a long line of work on **sample-efficient and model-based RL**, deep RL for continuous control, imitation/offline RL, and real-robot deployment. Senior author on PETS ([[deep-reinforcement-learning-handful-trials-using]]), the probabilistic-ensemble + trajectory-sampling MBRL algorithm that became the reference recipe for closing the asymptotic gap between model-based and model-free RL. The same lab's "learn to adapt fast" line — meta-learned online dynamics adaptation ([[learning-adapt-dynamic-real-world-environments]], Nagabandi et al.) — is the sibling deep-MBRL-with-MPC effort that names PETS-style probabilistic ensembles as its complementary uncertainty mechanism.

## Recent work

- [[deep-reinforcement-learning-handful-trials-using]] — PETS: probabilistic ensembles with trajectory sampling for sample-efficient model-based RL (senior author).
- [[planning-diffusion-flexible-behavior-synthesis]] — Diffuser: trajectory-level diffusion planning that folds modeling and planning into one denoising process, with cost/classifier guidance and inpainting as planning operators (senior author).

## My notes

For this wiki's rope/DLO arc, Levine is the senior-author node behind the **learned-model-plus-planning** backbone the principal leans on: PETS is the canonical uncertainty-aware MBRL algorithm and the principal's chosen primary defense against a planner exploiting an imperfect learned dynamics model. His group's broader MBRL/meta-RL output (PETS, GrBAL/ReBAL via Nagabandi) is the locomotion/continuous-control parent of the more specialized rope dynamics-and-planning methods elsewhere in the wiki.
