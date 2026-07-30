---
name: "Michael Janner"
affiliation: "UC Berkeley (EECS, PhD at time of this work)"
research_areas: [reinforcement-learning, model-based-reinforcement-learning, generative-models, planning, deep-learning]
homepage: "https://jannerm.github.io/"
scholar: "https://scholar.google.com/citations?user=BfDKO8EAAAAJ"
date_updated: 2026-06-16
type:
  kind: researcher
---

## Research areas

- [[model-based-planning-for-manipulation]]
Reinforcement learning and decision-making with a focus on **generative models as planners** — most prominently the line that treats sequence/trajectory generation and planning as the same operation. Co-first author on Diffuser ([[planning-diffusion-flexible-behavior-synthesis]]), the trajectory-level diffusion model that founded planning-as-diffusion, and earlier author of the Trajectory Transformer (offline RL as one big sequence-modeling problem), establishing a consistent thesis: fold the planning pipeline into the model so that sampling and planning coincide.

## Recent work

- [[planning-diffusion-flexible-behavior-synthesis]] — Diffuser: a trajectory-level denoising diffusion model that plans by guided sampling (return-gradient guidance + start/goal inpainting), founding the planning-as-diffusion formulation (co-first author).

## My notes

For this wiki's rope/DLO arc, Janner is the originating-author node behind **planning-as-diffusion** — the principal's chosen cost-guided trajectory-generation planner and the on-manifold structural defense against model exploitation. Diffuser is the direct ancestor of the deformable-object guided-diffusion line (e.g. DIDP in [[dynamic-manipulation-deformable-objects-3d-simulation]]), where the same recipe — a diffusion prior over feasible behavior steered by a cost/physics gradient at sampling time — is instantiated for 3D rope whipping.
