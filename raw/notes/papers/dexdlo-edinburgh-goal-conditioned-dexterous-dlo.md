---
title: "DexDLO: Learning Goal-Conditioned Dexterous Policy for Dynamic Manipulation of Deformable Linear Objects"
authors: "Sun Zhaole, Jihong Zhu, Robert B. Fisher"
venue: "ICRA 2024"
year: 2024
arxiv_id: "2312.15204"
doi: null
note_type: bibliography_only
sources: [report-1, report-2, report-3, l1b-1]
---

# DexDLO: Learning Goal-Conditioned Dexterous Policy for Dynamic Manipulation of Deformable Linear Objects

**One-line gist**: Model-free goal-conditioned RL on a fixed-base dexterous hand for DLO grabbing/pulling/end-tip placement in MuJoCo, abstracting multiple DLO tasks into a unified goal-conditioned formulation.

**Task setup**: Fixed-base anthropomorphic dexterous hand manipulates DLOs across five tasks including grabbing, pulling, and DLO end-tip position controlling. The policy is conditioned on a 3D goal point G and a tracked DLO point X, minimizing the distance between X and G dynamically without explicit regrasping or moving the hand base.

**Sim vs real**: Sim-only — no real-robot results reported in the verifiable source. Trained and evaluated entirely in MuJoCo.

**Learning method**: End-to-end model-free reinforcement learning with goal-conditioned reward. Five different tasks trained with the same hyperparameters, demonstrating framework generality.

**Action representation**: Joint-space control of the dexterous hand (fixed base). Observation is reduced (proprioceptive + DLO key-point) state.

**Why cited in the surveys**: One of the canonical examples of explicit 3D goal-conditioning for dynamic DLO manipulation, especially for end-tip targeting. Provides a useful conceptual reframing (dexterous hand instead of standard arm) but does not solve the standard arm + free rope-tip-to-target problem directly.

**Key result (if any)**: Five unified DLO tasks (including end-tip position control) learned with the same hyperparameters in MuJoCo simulation.
