---
title: "A Distributional Treatment of Real2Sim2Real for Object-Centric Agent Adaptation in Vision-Driven DLO Manipulation"
authors: "Georgios Kamaras, Subramanian Ramamoorthy"
venue: "IEEE RA-L Vol 10 Issue 8 Aug 2025 / arXiv (v4 Mar 2026)"
year: 2025
arxiv_id: "2502.18615"
doi: null
note_type: bibliography_only
sources: [report-2, l2a-2]
---

# A Distributional Treatment of Real2Sim2Real for Object-Centric Agent Adaptation in Vision-Driven DLO Manipulation

**One-line gist**: Likelihood-free inference computes posteriors over rope physical parameters; the posteriors then drive domain-randomized model-free RL for object-specific visuomotor policies.

**Task setup**: DLO reaching task; object-centric — different DLOs (different parameters) get separate posterior-conditioned policies.

**Sim vs real**: Zero-shot sim-to-real.

**Learning method**: Model-free RL + LFI-driven domain randomization. Distributional parameter treatment instead of single-point system ID.

**Action representation**: Visuomotor policy from vision + proprioception (specific action space not specified).

**Why cited in the surveys**: Methodology bridge — shows how Bayesian-posterior domain randomization (vs deterministic sysID) handles unknown rope parameters for sim2real reaching. Directly applicable to whip-to-target where rope mass/stiffness varies.

**Key result (if any)**: Zero-shot real deployment on DLO reaching; distinguishes parameterized DLOs from dynamic trajectory data.
