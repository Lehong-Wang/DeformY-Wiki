---
title: "A Learning-Based Control Framework for Human-Like Whip Targeting (also published as 'A Learning-based Control Framework for Fast and Accurate Manipulation of a Flexible Object')"
authors: "Wang, Xiong"
venue: "IEEE BioRob 2024 / Journal of Bionic Engineering 21:1761–1774, 2024"
year: 2024
arxiv_id: null
doi: "10.1007/s42235-024-00534-2"
note_type: bibliography_only
sources: [report-1]
---

# A Learning-Based Control Framework for Human-Like Whip Targeting

**One-line gist**: Learned/optimized motion planner + Online Impedance Adaptation Control (OIAC) + sim2real + visual feedback for fast (<1.5 s) accurate whip targeting; companion / sister paper to the SDU SIMPAR 2025 DRL whip-targeting work.

**Task setup**: Fast (<1.5 s), accurate whip-tip targeting in 3D. Pipeline = motion planner (learned/optimized) + OIAC for compliance on real hardware + visual feedback for closed-loop correction.

**Sim vs real**: Combination — uses MuJoCo to plan, OIAC for compliance on real hardware, visual feedback for closed-loop correction.

**Learning method**: Learned/optimized motion planner with adaptive impedance; explicitly inspired by the Sternad/Hogan dynamic-primitives perspective on human whip control.

**Action representation**: Parameterized arm motions, compatible with a dynamic-primitive framing.

**Why cited in the surveys**: Companion to Bai et al. (SIMPAR 2025) and the source of the OIAC sim-to-real pipeline they reuse. The most explicitly human-inspired robotics whip-targeting work, building on the Sternad/Hogan/Northeastern-MIT line.

**Key result (if any)**: Sim-vs-real trajectory similarity ~89% on whip-handle motion; OIAC reduces tracking error 13–22% vs constant-impedance baseline.
