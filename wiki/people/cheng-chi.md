---
name: "Cheng Chi"
affiliation: "Columbia University (at time of publication); Stanford University (subsequent)"
tags: [robot-learning, manipulation, deformable-objects, diffusion-policy, dynamic-manipulation]
homepage: "https://cheng-chi.github.io/"
scholar: "https://scholar.google.com/citations?user=4juDyLIAAAAJ"
date_updated: 2026-05-06
---

## Research areas

- [[dynamic-dlo-tip-targeting]]
- Goal-conditioned manipulation of deformable objects (ropes, cloths)
- Dynamic manipulation and momentum-exploiting motion primitives
- Sim-to-real transfer via online adaptation rather than randomization
- Generative robot policies (diffusion policies)
- Iterative residual learning frameworks

## Key papers

- [[iterative-residual-policy-goal-conditioned-dynamic]] (RSS 2022 Best Paper; IJRR 2024) — first author. Introduces Iterative Residual Policy (IRP) for goal-conditioned dynamic manipulation of ropes and cloths, demonstrating zero-shot sim-to-real generalization.

## Recent work

- [[iterative-residual-policy-goal-conditioned-dynamic]] — IRP (RSS 2022 Best Paper / IJRR 2024), first author: learned delta-dynamics over a low-dimensional swing primitive with iterative action refinement; zero-shot sim-to-real rope whipping.
- [[diffusion-policy-visuomotor-policy-learning-action]] — Diffusion Policy (RSS 2023; IJRR 2024), joint first author. Conditional denoising diffusion over action chunks with receding-horizon execution; +46.9% average over prior BC methods across 15 tasks in 4 benchmarks.

## Collaborators

- Shuran Song (PhD advisor at Columbia) — see [[shuran-song]]
- Benjamin Burchfiel, Eric Cousineau, Siyuan Feng (Toyota Research Institute) — co-authors on IRP
- Toyota Research Institute (TRI) — early career industry collaboration

## My notes

Cheng Chi is the architect of the iterative-residual-policy formulation. His contributions to the line of dynamic deformable manipulation (IRP) and later diffusion policies are central to the modern manipulation literature. For DeformY, IRP is the anchor reference; Chi's continued work on sim-to-real and policy classes is worth tracking when subsequent papers are ingested.
