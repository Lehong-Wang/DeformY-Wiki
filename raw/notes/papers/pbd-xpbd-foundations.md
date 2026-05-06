---
title: "Position-Based Dynamics / XPBD Foundations + Differentiable PBD for Rope Manipulation (Liu / Müller / Macklin)"
authors: "Matthias Müller (Müller et al. 2007); Miles Macklin & Matthias Müller (XPBD 2016); Y.-J. Liu, Y. Su, R. Li, Z. Ji, M. Yip et al. (arXiv 2202.09714)"
venue: "Foundational papers (2007, 2016) + arXiv 2022"
year: 2007
arxiv_id: "2202.09714"
doi: null
note_type: bibliography_only
sources: [report-1]
---

# PBD / XPBD Foundations + Differentiable PBD for Rope Manipulation

**One-line gist**: Cluster of foundational PBD/XPBD references + the Liu et al. 2022 differentiable XPBD-based rope manipulator.

**Task setup**: Foundational simulator formulations + a downstream differentiable rope simulator with parameter ID, demonstrated on Baxter and dVRK platforms.

Cluster:
- Müller et al., "Position-Based Dynamics" (2007). Original PBD formulation.
- Macklin & Müller, "XPBD" (2016). Extended PBD that decouples stiffness from iteration count.
- Liu, Su, Li, Ji, Yip et al., "Differentiable Robotic Manipulation of Deformable Rope-like Objects Using Compliant Position-based Dynamics" (arXiv 2202.09714, 2022). XPBD-based differentiable simulator with parameter ID.

**Sim vs real**: Foundational papers are sim-only. Liu et al. 2022 demonstrates real Baxter and dVRK manipulation with parameter-ID via the differentiable XPBD simulator.

**Learning method**: None for foundations. Liu et al. 2022 uses differentiable physics for parameter ID.

**Action representation**: N/A for foundations. Liu et al. 2022 uses standard robot control commands.

**Why cited in the surveys**: PBD/XPBD are the dominant fast-but-physically-approximate alternative to DER/Cosserat for rope simulation. The Liu et al. 2022 differentiable XPBD rope simulator is cited in Report 1 as the canonical PBD-flavored differentiable rope manipulator. Reports note PBD/XPBD are *not* validated for whip-cracking high-curvature regimes — DER is preferred there.

**Key result (if any)**: Liu et al. 2022 demonstrates differentiable XPBD-based rope manipulation with parameter ID on real hardware.
