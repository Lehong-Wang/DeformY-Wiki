---
title: "Dynamic Manipulation of Deformable Objects with Implicit Integration"
authors: "Simon Zimmermann, Roi Poranne, Stelian Coros"
venue: "IEEE RA-L + ICRA 2021"
year: 2021
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1, report-3]
---

# Dynamic Manipulation of Deformable Objects with Implicit Integration

**One-line gist**: Model-based optimal control (Newton/DDP) through an implicit-integration deformable simulator with analytic gradients, applied to dual-arm dynamic manipulation including whip-like striking of target objects.

**Task setup**: A dual-arm YuMi robot grasps one end of a deformable cable/cloth and produces dynamic motions to achieve desired configurations. One showcased task uses a soft rod as a whip to hit target objects; other tasks define goals via target positions of selected points over time. Goal is a configuration / point-trajectory, not a single 3D tip point per se.

**Sim vs real**: Mostly simulation; analytic derivatives through implicit integration enable model-based control. Real-world demos included.

**Learning method**: None — model-based optimal control. Compares batch Newton and DDP under a soft-body simulator with implicit integration (BDF2 time-stepping).

**Action representation**: Open-loop end-effector / joint-space trajectory optimized via DDP / batch Newton against a differentiable deformable-body model.

**Why cited in the surveys**: The strongest classical/optimization baseline for "whip a soft body to hit a target." Uses an implicit-integration FEM-like soft-body sim that is differentiable, predating but closely related to differentiable DLO simulators like DEFORM. Cited as the main non-learning benchmark for whip-to-target.

**Key result (if any)**: Demonstrated whip striking of target objects + several dynamic soft-body tasks on real YuMi using model-based optimal control.
