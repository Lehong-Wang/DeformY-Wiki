---
title: "Differentiable Physics Simulations with Contacts: Do They Have Correct Gradients?"
authors: "Y. Zhong, J. Han, G. O. Brikis"
venue: "arXiv 2207.05060"
year: 2022
arxiv_id: "2207.05060"
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Zhong, Han, Brikis — Differentiable Physics Simulations with Contacts: Do They Have Correct Gradients?

**One-line gist**: Survey-style analysis of which differentiable physics simulators produce correct gradients in the presence of contacts; an essential reference for choosing a simulator for whipping.

**Task setup**: Comparative analysis of differentiable contact gradients across DiffTaichi, Brax, Warp, Tiny Differentiable Simulator, Dojo, DiffCoSim, etc.

**Sim vs real**: N/A (analysis).

**Learning method**: None.

**Action representation**: N/A.

**Why cited in the surveys**: Cited in Report 1 as the essential reference for selecting a differentiable physics back-end when policy gradients matter — including for differentiable-physics-trained whip policies.

**Key result (if any)**: Identifies which differentiable simulators have correct contact gradients vs which approximate them.
