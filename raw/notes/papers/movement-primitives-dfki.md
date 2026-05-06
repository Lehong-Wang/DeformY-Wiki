---
title: "dfki-ric/movement_primitives — DMP / ProMP / Bimanual Coupled-DMP Toolkit"
authors: "DFKI Robotics Innovation Center (RIC)"
venue: "Open-source library"
year: 2025
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-2, l2d-5]
---

# dfki-ric/movement_primitives

**One-line gist**: Best-maintained open-source library for Dynamic Movement Primitives (DMPs), Probabilistic Movement Primitives (ProMPs), and spatially-coupled bimanual DMPs in Cartesian space.

**Task setup**: Library, not a single paper. Cython for online-execution speed. 290 stars, BSD-3, last release v0.9.1 (May 2025).

**Sim vs real**: Library — used in both contexts.

**Learning method**: Provides DMP/ProMP regression and adaptation infrastructure; supports goal-conditioned and bimanual settings.

**Action representation**: DMP shape + goal modulation.

**Why cited in the surveys**: The whip-tip-targeting line (Krotov, Nah, Edraki) repeatedly uses DMP-style motor primitives as the action representation. This is the right "starter kit" for goal-conditioned DMP training in this problem.

**Key result (if any)**: Production-grade DMP toolkit. Other DMP repos surveyed: stulp/dmpbbo (DMP + CMA-ES), studywolf/pydmps, AlexanderFabisch/PyDMP.
