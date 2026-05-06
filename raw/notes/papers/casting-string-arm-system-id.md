---
title: "Casting Manipulation of Unknown String by Robot Arm + Motion Planning for Dynamic Three-Dimensional Manipulation for Unknown Flexible Linear Object"
authors: "(2021 IROS + 2024 extension)"
venue: "IROS 2021 / 2024 follow-up"
year: 2021
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-3]
---

# Casting Manipulation of Unknown String by Robot Arm (and 2024 3D extension)

**One-line gist**: Classical system-ID-heavy baseline for casting an unknown string with a normal robot arm; alternates motion generation, manipulation, and parameter estimation until simulator matches reality. 2024 follow-up extends to 3D.

**Task setup**: A standard robot arm dynamically casts a flexible string (instead of a custom casting manipulator). The 2021 IROS paper alternates among (i) motion generation, (ii) actual string manipulation, and (iii) string-parameter estimation until the simulator matches reality closely enough to cast the string toward the target. The 2024 extension explicitly extends the same approach into 3D, targeting desired objects with the flexible linear object's tip.

**Sim vs real**: Real-robot system-ID + sim-based planning; classical loop.

**Learning method**: None — classical system identification + simulation-based planning.

**Action representation**: Casting trajectory parameters; iteratively refined via system ID.

**Why cited in the surveys**: Cited in Report 3 as the cleanest classical analog of "dynamic 3D string casting with parameter estimation" — a system-ID-heavy baseline against which learned policies (IRP, IPA) can be compared. The 2024 extension is one of the few classical 3D tip-target casting systems before the modern wave.

**Key result (if any)**: Successfully cast unknown strings to 3D targets through the iterative motion-generation/system-ID loop.
