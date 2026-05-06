---
title: "Self-Supervised Physics-Informed Manipulation of DLOs (SPiD)"
authors: "Long, Solak, Zeynalpour, Zhang, Ajoudani"
venue: "arXiv 2602.03623"
year: 2025
arxiv_id: "2602.03623"
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Self-Supervised Physics-Informed Manipulation of DLOs (SPiD)

**One-line gist**: Differentiable mass-spring rope simulator + neural controller trained self-supervised via task-oriented cost back-propagated through the differentiable physics; deployed on real DLOs for stabilization/shape control.

**Task setup**: Dynamic stabilization / shape control of ropes (rope-stabilization task shown). Not pure tip-targeting, but uses dynamic-regime motions.

**Sim vs real**: Differentiable simulator with system ID; controller trained via self-supervision in differentiable physics; deployed real.

**Learning method**: Self-supervised training of a neural controller via task-oriented cost back-propagated through a differentiable mass-spring rope model.

**Action representation**: Direct neural-controller output (joint or end-effector velocity commands).

**Why cited in the surveys**: 2025-era exemplar of "differentiable physics + self-supervised neural controller" recipe for dynamic DLO control. Adjacent to the dynamic whip-tip line — uses dynamic motions but conditions on stability/shape rather than a 3D tip target.

**Key result (if any)**: Real-world rope stabilization demonstrated.
