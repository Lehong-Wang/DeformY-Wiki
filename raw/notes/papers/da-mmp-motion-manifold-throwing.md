---
title: "DA-MMP: Dynamics-Aware Motion Manifold Primitives"
authors: "Chu, Xu"
venue: "arXiv 2509.23721"
year: 2025
arxiv_id: "2509.23721"
doi: null
note_type: bibliography_only
sources: [report-1]
---

# DA-MMP: Dynamics-Aware Motion Manifold Primitives

**One-line gist**: Goal-conditioned ring-tossing on a real 7-DoF arm; learns a low-dimensional manifold of feasible variable-length throwing trajectories via conditional flow-matching, fine-tuned with a small real dataset to bridge the dynamics gap.

**Task setup**: Goal-conditioned ring-tossing on a real 7-DoF robot arm. Goal = 3D target.

**Sim vs real**: Mostly real-world; simulation used for trajectory collection. Small real dataset for fine-tuning.

**Learning method**: Conditional flow-matching model in latent space encoding a low-dimensional manifold of feasible throwing trajectories. The latent decoder produces variable-length trajectories.

**Action representation**: Variable-length parametric trajectories on a learned latent manifold.

**Why cited in the surveys**: One of the leading 2025 goal-conditioned throwing analogs that explicitly uses a *latent motion-manifold* action representation — different from scalar/release-velocity (TossingBot) and full-trajectory diffusion. Methodologically transferable to whip-handle motion.

**Key result (if any)**: Goal-conditioned ring-tossing on a real 7-DoF arm with sim-to-real fine-tuning.
