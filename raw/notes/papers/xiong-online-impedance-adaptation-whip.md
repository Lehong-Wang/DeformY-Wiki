---
title: "Online Impedance Adaptation Facilitates Manipulating a Whip (OIAC)"
authors: "Xiong, Nah, Krotov, Sternad"
venue: "Sternad/Hogan whip-control series"
year: 2023
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Xiong et al. — Online Impedance Adaptation Facilitates Manipulating a Whip

**One-line gist**: Adaptive impedance control framework (OIAC) that improves whip-manipulation tracking; the OIAC primitive used by the Bai/Wang/Xiong SIMPAR-2025 sim-to-real pipeline.

**Task setup**: Whip manipulation via an adaptive-impedance controller; the OIAC primitive sets compliance online during high-speed swings.

**Sim vs real**: OIAC was used as the sim-to-real bridge in subsequent robotics whip-targeting work (Bai/Wang/Xiong 2024–2025).

**Learning method**: Online adaptive impedance — not RL. Adapts joint-stiffness / damping during the swing.

**Action representation**: Joint-space impedance commands.

**Why cited in the surveys**: Cited in Report 1 as the source of the OIAC primitive that the Bai/Wang/Xiong robotics group (SIMPAR 2025, BioRob 2024) reuses for sim-to-real. Part of the Sternad/Hogan/Northeastern–MIT cluster of whip-control works.

**Key result (if any)**: OIAC reduces tracking error for fast whip motions vs constant-impedance baselines; downstream robotics whip-targeting reports 13–22% reduction.
