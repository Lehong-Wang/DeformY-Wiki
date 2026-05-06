---
title: "GenORM: Generalizable One-shot Rope Manipulation with Parameter-Aware Policy"
authors: "(See arXiv 2306.09872)"
venue: "CoRL/ICRA 2024"
year: 2023
arxiv_id: "2306.09872"
doi: null
note_type: bibliography_only
sources: [report-1]
---

# GenORM: Generalizable One-shot Rope Manipulation with Parameter-Aware Policy

**One-line gist**: Parameter-aware policy + differentiable physics ID from a single real demo, enabling one-shot generalization across ropes with different physics; quasi-static.

**Task setup**: Quasi-static rope manipulation tasks where the policy must generalize across ropes with different stiffness, mass, and friction. The system uses one real demo to identify rope parameters.

**Sim vs real**: Differentiable physics for simulation-side parameter ID; real-robot for one-shot demonstration and deployment.

**Learning method**: Parameter-aware policy conditioned on physics parameters extracted by differentiable simulation from a single real demo.

**Action representation**: Quasi-static manipulation primitives (typically pick-place / EE-trajectory).

**Why cited in the surveys**: Methodological precursor to "implicit physics-aware" / "system-ID then act" recipes (IPA, Wiggle and Go!) for rope manipulation. Cited in Report 1 as a foundational generalizable, parameter-aware DLO method, even though it is quasi-static.

**Key result (if any)**: One-shot generalization across ropes from a single real demo.
