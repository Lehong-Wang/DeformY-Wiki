---
title: "Rearranging Deformable Linear Objects for Implicit Goals with Self-Supervised Planning and Control"
authors: "S. Huo, F. Hu, F. Wang, L. Hu, P. Zhou, J. Zhu, H. Wang, D. Navarro-Alarcón"
venue: "Advanced Intelligent Systems (Wiley) 2025"
year: 2025
arxiv_id: null
doi: "10.1002/aisy.202400330"
note_type: bibliography_only
sources: [report-2, l1b-6]
---

# Rearranging Deformable Linear Objects for Implicit Goals with Self-Supervised Planning and Control

**One-line gist**: Implicit-goal DLO rearrangement: dual-arm robot rearranges a DLO so both ends are reachable/graspable, generating its own explicit candidate targets that satisfy the implicit constraint.

**Task setup**: A user specifies an implicit reachability/graspability constraint rather than an explicit target shape. The system designs a compact descriptor for the DLO state under physical constraints, generates multiple explicit candidate targets that satisfy the implicit condition, evaluates achievability, and executes the optimal one. Dual-arm robot.

**Sim vs real**: Both, per the published article.

**Learning method**: Self-supervised planning + control with learned achievability models. No human demonstrations or hand-labels.

**Action representation**: Motion primitives constrained by a learned achievability model.

**Why cited in the surveys**: Distinctive *implicit-goal* formulation for DLO manipulation — the target is a constraint over reachability rather than a tip-point or shape. Useful as an alternate target-encoding paradigm to compare with point/shape/language conditioning.

**Key result (if any)**: Demonstrates implicit-goal rearrangement of DLOs with self-supervised data collection on real hardware.
