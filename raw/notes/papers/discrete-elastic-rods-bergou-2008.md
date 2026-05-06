---
title: "Discrete Elastic Rods (DER)"
authors: "Miklós Bergou, Max Wardetzky, Stephen Robinson, Basile Audoly, Eitan Grinspun"
venue: "ACM SIGGRAPH 2008 (TOG 27(3))"
year: 2008
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1, report-2, report-3]
---

# Discrete Elastic Rods (DER) — Bergou et al. 2008

**One-line gist**: Foundational discrete-geometric model of thin flexible rods with bending and twisting (Bishop-frame angle as material-frame DoF, quasi-static frame, dynamic centerline); the basis for DEFORM, DEFT, DisMech, and ~all modern rope-simulation work in robotics.

**Task setup**: A simulation/modeling formulation, not a robot task. Validates buckling, stability, coupled-mode behavior, and knot-tying.

**Sim vs real**: Sim-only.

**Learning method**: None.

**Action representation**: N/A.

**Why cited in the surveys**: Cited as the foundational reference for discrete elastic rods in computer graphics / mechanics — the substrate for DEFORM, DEFT, DisMech, MAT-DiSMech, and DER-MuJoCo. Companion paper "Discrete Viscous Threads" (Bergou et al. 2010, SIGGRAPH) extends to viscous rod-like fluids.

**Key result (if any)**: Established a quaternion-free Bishop-frame discretization that captures bending, twisting, and inertia with high fidelity at modest cost.
