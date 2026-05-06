---
title: "Human-Inspired Control of a Whip: Preparatory Movements Improve Hitting a Target"
authors: "Mahdiar Edraki, Rajiv Lokesh, Aleksei Krotov, Alireza Ramezani, Dagmar Sternad"
venue: "IEEE BioRob 2024 (also ICRA 2025 RMDO spotlight)"
year: 2024
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1, report-2, l1d-3]
---

# Edraki et al. 2024 — Preparing-and-Striking Trajectory Optimization for Whip Target Reaching

**One-line gist**: Forward-dynamics Lagrangian model of a 25-link 1 m whip + PD joint stiffness/damping; MATLAB pattern-search over 5-param "striking-only" vs 9+1-param "preparing-and-striking" minimum-jerk handle trajectories.

**Task setup**: 1.0 m, 25 spherical-link physical whip (40 mm dia, ~7 g each) connected by steel pin hinges. Target distance 1.2 m horizontal in front of the handle; two heights (0.1 m above and 0.2 m below handle initial position). 5 human volunteers (ages 23–30) reproducing planned trajectories on a 3D-printed physical whip. Hit threshold 1 cm.

**Sim vs real**: Sim (forward-dynamics) + small physical-whip validation. Model error vs physical < 5 cm tip RMS.

**Learning method**: Trajectory optimization over a fixed motor-primitive parameter family. MATLAB `patternsearch`, max 200 iters, mesh tol 1e-4. Forward dynamics from analytical Lagrangian model; PD low-level joint control. **No RL, no neural networks.**

**Action representation**: Two control strategies:
- *Striking-only*: single planar minimum-jerk handle trajectory parameterized by 5 vars (`x_i, z_i, x_f, z_f, t_f`).
- *Preparing-and-striking*: two sequential minimum-jerk segments + temporal overlap = 9 position/duration vars + 1 overlap percentage.

**Why cited in the surveys**: Direct robotic / physical instantiation of the Nah 2023 simulation insight that a single smooth primitive can hit and a preparatory segment extends reach. Provides an explicit numerical-optimization formulation that any learning-based whip paper compares against as a model-based baseline.

**Key result (if any)**: Striking-only reaches only near targets; preparing-and-striking significantly extends reach (1.2 m far targets reachable only with preparatory backward submovement). 1 cm hit threshold achieved.
