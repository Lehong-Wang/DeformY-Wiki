---
name: "Chi Chu"
affiliation: "Shanghai Qi Zhi Institute"
research_areas: [dynamic-manipulation, motion-manifolds, movement-primitives, trajectory-generation, kinodynamic-planning, robot-learning]
scholar: "https://www.semanticscholar.org/author/2355235575"
date_updated: 2026-07-30
type:
  kind: researcher
---

## Research areas

Motion generation for dynamic manipulation: representing high-speed whole-arm motions compactly enough to learn over, and closing the planned-vs-executed dynamics gap without an explicit dynamics model. First author of DA-MMP, which bridges the motion-manifold-primitives line (representation) and the goal-conditioned dynamic-manipulation line (dynamics gap) by learning a manifold from 90k planner-generated trajectories and a latent conditional flow-matching model from 60 real trials labeled by their measured outcomes. Practical footprint spans kinodynamic sampling-based planning (DIMT-RRT, bounded-acceleration shortcut smoothing), via-point movement primitives, and flow-matching generative models.

## Recent work

- [[da-mmp-learning-coordinated-accurate-throwing]] — DA-MMP (arXiv 2509.23721, 2025): variable-length motion manifold primitives + latent conditional flow matching on executed landing outcomes; 60% real-world ring-toss success, above trained human experts.

## My notes

Small publication record (S2: 2 papers as of ingest), so the profile is essentially this one paper — but it is the paper closest to the rope-swing project's chosen recipe, and worth watching for a follow-up. The acknowledgments credit Kris Hauser and Tobias Kunz for kinodynamic-planning discussions, which is consistent with the corpus-generation stack being the load-bearing engineering. No code released; contact address is in the paper (`cc2018011855@gmail.com`) if a replication attempt needs the parameterization details, in particular the corrected time-to-phase coefficient the paper's own footnote flags.
