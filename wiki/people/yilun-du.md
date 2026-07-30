---
name: "Yilun Du"
affiliation: "MIT (CSAIL, PhD at time of this work); Harvard / Google DeepMind (subsequent)"
research_areas: [generative-models, energy-based-models, diffusion-models, compositional-generation, reinforcement-learning, robot-learning]
homepage: "https://yilundu.github.io/"
scholar: "https://scholar.google.com/citations?user=Bm_jBUEAAAAJ"
date_updated: 2026-06-16
type:
  kind: researcher
---

## Research areas

Generative modeling with an emphasis on **energy-based models, diffusion models, and compositionality** — composing learned distributions (and their gradients) to synthesize novel behaviors and images. Co-first author on Diffuser ([[planning-diffusion-flexible-behavior-synthesis]]), where the compositional view of guided diffusion sampling — adding multiple perturbation gradients to satisfy several objectives/constraints at once — is the mechanism behind its task and temporal compositionality. His broader EBM/compositional-generation line directly motivates the cost-as-perturbation framing at the heart of planning-as-diffusion.

## Recent work

- [[planning-diffusion-flexible-behavior-synthesis]] — Diffuser: trajectory-level diffusion planning; the compositional guided-sampling formulation (summing reward/constraint gradients) underlies its multi-task and goal-conditioned flexibility (co-first author).

## My notes

Du is the generative-modeling-and-compositionality node behind **planning-as-diffusion**: the reinterpretation of classifier guidance and inpainting as composable planning operators is the bridge from his EBM/compositional-generation work to behavior synthesis. For the DeformY arc this matters because cost-guided diffusion planning over rope behavior (DIDP-style) inherits exactly this compositional cost-as-guidance structure.
