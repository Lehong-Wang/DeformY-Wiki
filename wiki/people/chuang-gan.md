---
name: "Chuang Gan"
affiliation: "University of Massachusetts Amherst; MIT-IBM Watson AI Lab"
research_areas: [embodied-AI, differentiable-simulation, robot-learning, multimodal-learning, physics-based-scene-understanding, deformable-object-manipulation]
homepage: "https://people.csail.mit.edu/ganchuang/"
scholar: "https://scholar.google.com/citations?user=PTeSCbIAAAAJ"
date_updated: 2026-07-30
type:
  kind: researcher
---

## Research areas

- Embodied AI and simulation platforms (Genesis, ThreeDWorld, FluidLab, PlasticineLab lineage)
- Differentiable physics for policy learning and system identification
- Deformable-object manipulation (fluids, elastoplastics, and now deformable linear objects)
- Multimodal and neuro-symbolic representation learning
- Physics-based scene understanding from video

## Recent work

- [[dlo-lab-benchmarking-deformable-linear-object]] (ICML 2026) — senior author. Differentiable
  DLO simulator + 10-task benchmark; the analysis that gradient-free CMA-ES trajectory
  optimization outperforms analytic-gradient and model-free RL methods on contact-mediated DLO
  tasks.

## My notes

The through-line across this group's simulation work — PlasticineLab (elastoplastics), FluidLab
(fluids), Genesis (general-purpose), and now DLO-Lab (rods) — is that each release adds a material
class to a differentiable, Taichi-backed platform and then benchmarks gradient-based against
gradient-free optimization on it. Worth tracking for this wiki mainly as the supply line for the
independent verifier stack; the group's own conclusions about when differentiability pays are the
best-calibrated external evidence available for the project's gated
[[sim-stage-d-gated-extensions]] D5 item, and they are more negative than the differentiable-physics
literature's usual framing.
