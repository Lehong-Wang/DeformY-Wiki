---
title: "Learning to Manipulate a Whip with Simple Primitive Actions — A Simulation Study"
authors: "Moses C. Nah, Aleksei Krotov, Marta Russo, Dagmar Sternad, Neville Hogan"
venue: "iScience 26(8):107395"
year: 2023
arxiv_id: null
doi: "10.1016/j.isci.2023.107395"
note_type: bibliography_only
sources: [report-1, report-2, l1d-2]
---

# Nah et al. 2023 — Whip Primitives + DIRECT-L Optimization for Tip-Targeting

**One-line gist**: 4-DoF arm + 50-DoF segmented whip in MuJoCo; classical black-box trajectory optimization (DIRECT-L from NLopt) over a 9-parameter minimum-jerk joint-space primitive reaches all 6 of 6 3D targets in 39–249 iterations.

**Task setup**: 4-DOF arm (3 shoulder rotations + elbow flexion) + 50-DOF segmented whip (25 sub-models serially connected, segmented elastic chain) in MuJoCo with semi-implicit Euler at 0.1 ms. Gravity included with compensation torque. Total system 54-DOF underactuated. 6 3D targets in spherical coordinates (3 within reach + 3 at maximum reach); target = 3 cm-radius sphere.

**Sim vs real**: Sim-only (MuJoCo).

**Learning method**: Classical black-box trajectory optimization (DIRECT-L from NLopt — deterministic Lipschitzian, derivative-free) over a 9-D continuous parameter vector. Cost = closest tip-to-target approach distance. NOT RL.

**Action representation**: 9-parameter minimum-jerk discrete movement in joint space — movement duration `D` (1) + initial joint posture `q0,i` ∈ ℝ⁴ (4) + final joint posture `q0,f` ∈ ℝ⁴ (4). Bell-shaped speed profile matching human discrete movements.

**Why cited in the surveys**: The biggest-bang-for-buck classical baseline for any RL/learning whip-targeting paper. Demonstrates the underactuation barrier is dimensional, not control-theoretic — a 9-D smooth primitive subspace is enough to hit 3D targets with a 50-DOF whip with no closed-loop feedback, no learning, no RL.

**Key result (if any)**: Single submovement reached or closely approached all 6 targets in 39–249 DIRECT-L iterations (max 600 allowed). Code: https://github.com/MosesAndLily/whip-project-targeting (MuJoCo, Python, NLopt; BSD-3).
