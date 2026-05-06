---
title: "As of May 2026, no published paper combines all four of {real-robot hardware, arbitrary 3D target, learned runtime policy, free-space dynamic whipping} for rope-tip targeting"
slug: "no-paper-has-real-3d-arbitrary-target-learned-policy-whipping"
status: weakly_supported
confidence: 0.7
tags: [DLO, dynamic-manipulation, rope-tip-targeting, gap-analysis, research-direction]
domain: "Robotics"
date_updated: "2026-05-06"
source_papers:
  - iterative-residual-policy-goal-conditioned-dynamic
  - dynamic-manipulation-deformable-objects-3d-simulation
  - wiggle-go-system-identification-zero-shot
  - implicit-physics-aware-policy-dynamic-manipulation
  - learning-deformable-object-manipulation-using-task
evidence:
  - source: iterative-residual-policy-goal-conditioned-dynamic
    type: contradicts_partially
    note: "IRP achieves real-robot + learned runtime policy + dynamic whipping, but its verified target space is 2D Y–Z plane, not arbitrary 3D."
  - source: dynamic-manipulation-deformable-objects-3d-simulation
    type: contradicts_partially
    note: "DIDP achieves arbitrary 3D + learned runtime policy + dynamic whipping, but is sim-only — no real-hardware validation."
  - source: wiggle-go-system-identification-zero-shot
    type: contradicts_partially
    note: "Wiggle&Go achieves real-robot + arbitrary 3D + dynamic whipping, but uses sysID + CMA-ES trajectory optimization in Drake at task time rather than a learned runtime policy."
  - source: implicit-physics-aware-policy-dynamic-manipulation
    type: contradicts_partially
    note: "IPA achieves real-robot + learned runtime policy with rope-as-tool, but the goal is rigid-payload landing, not free-tip targeting."
  - source: learning-deformable-object-manipulation-using-task
    type: contradicts_partially
    note: "Task-Level ILC achieves real-robot + dynamic, but the target is a topological state (flying knot), not arbitrary 3D point; and ILC is iterative across trials per goal."
---

## Statement

As of 2026-05-06, the published literature does not yet contain a single paper that simultaneously delivers all four properties for rope-tip targeting:

1. **Real robot hardware** (not just simulation)
2. **Arbitrary 3D target point** in free space (not 2D plane, not topological state, not single-shot proxies)
3. **Learned runtime policy** (not classical sysID + trajectory optimization at task time)
4. **Free-space dynamic whipping** (not quasi-static, not fixed-endpoint cable, not heterogeneous payload landing)

Each property is individually realized; the combination is the niche.

## Conditions

- The claim is restricted to the published literature surveyed in 2026-05-06 from three independent surveys (Claude, agentic-search, GPT).
- "Learned runtime policy" excludes pipelines whose action-time computation is dominated by classical optimization (CMA-ES, DDP, Newton, MPC over an analytic model).
- "Arbitrary 3D target" excludes papers whose target is a 2D plane projection, a binary task-success bit, a topological state (knot tied), or a configuration distance to a goal shape.

## Evidence

The four most-cited candidate papers each fail one and only one property:

| Paper | Real-robot | Arbitrary 3D | Learned policy | Whipping (vs other) | Failure axis |
|---|---|---|---|---|---|
| [[iterative-residual-policy-goal-conditioned-dynamic]] (Chi RSS 2022) | ✓ UR5+Sawyer | **2D Y–Z plane** | ✓ delta-dynamics network | ✓ rope-tip whip | 3D |
| [[dynamic-manipulation-deformable-objects-3d-simulation]] (Lan 2025, DIDP) | **sim only** | ✓ 3D rope tip | ✓ diffusion policy + PITA | ✓ rope tip in air | real-robot |
| [[wiggle-go-system-identification-zero-shot]] (CMU 2026) | ✓ xArm 7 | ✓ 3D rope tip | **CMA-ES trajopt at task time** | ✓ rope-tip strike | learned policy |
| [[implicit-physics-aware-policy-dynamic-manipulation]] (Wang & Qureshi 2025) | ✓ UR5e | ✓ 3D | ✓ ResNet velocity regressor | **rigid payload land, not free-tip** | task formulation |
| [[learning-deformable-object-manipulation-using-task]] (Suresh CMU 2026) | ✓ xArm 7 | **topological state** | ✓ ILC across trials | ✓ dynamic | 3D-target |

## Counter-evidence and conditions for revision

- A 2026-onward arXiv preprint that closes the gap would invalidate the claim. Watch for: (i) a real-hardware deployment of DIDP or its successors, (ii) a learned-policy-on-Drake follow-up to Wiggle-and-Go that replaces CMA-ES with a learned action head, (iii) a free-tip extension of IPA.
- The claim is robust to alternate definitions of "whipping" (e.g., loose vs taut rope, sub-supersonic vs supersonic tip) — every paper surveyed operates well sub-supersonic, so the gap is not about whip-physics fidelity.
- The 2026 preprint surge (SoftMimicGen, RopeDreamer, RAPiD, Self-Curriculum-MBRL) is consolidating the data-generation, world-modeling, and adaptation infrastructure that makes the gap closable; expect revision within 12 months.

## Why it matters

This is the cleanest single-sentence research-direction statement extractable from the 3-survey synthesis. A research project that targets exactly this niche — pre-print Wiggle-and-Go's hardware setup, replace its CMA-ES head with a learned diffusion policy initialized by IRP and adapted via DIDP-style PITA — would be the natural successor paper.

## Linked ideas

(empty — populate when an `/ideas/` page proposes attacking this gap.)
