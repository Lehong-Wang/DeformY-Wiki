---
title: "Yamakawa-Namiki-Ishikawa High-Speed Flexible Robotics Series (Univ. Tokyo, 2007–2019)"
authors: "Yuji Yamakawa, Akio Namiki, Taku Senoo, Masatoshi Ishikawa"
venue: "Multiple — IROS 2010, ICRA 2012, IJARS 2013, ICRA 2013, IntechOpen 2019"
year: 2010
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1, report-3]
---

# Yamakawa-Namiki-Ishikawa High-Speed Flexible Robotics Series

**One-line gist**: The foundational classical robotics line on dynamic high-speed rope/ribbon/cloth manipulation: 1 kHz vision + 4-DOF Barrett WAM at very high speed + simplified algebraic deformation models valid in the high-speed regime.

**Task setup**: Across the series — high-speed knot tying of flexible rope, dynamic ribbon-shape generation for rhythmic gymnastics, dynamic cloth folding, casting of unknown strings. Tasks are framed by motion/topology success (knot formation, ribbon configuration), not 3D tip-position targets.

Major papers in the series:
- Yamakawa, Namiki, Ishikawa, "Motion Planning for Dynamic Knotting of a Flexible Rope with a High-Speed Robot Arm" (IROS 2010).
- Yamakawa, Namiki, Ishikawa, "Simple Model and Deformation Control of a Flexible Rope using Constant, High-Speed Motion of a Robot Arm" (ICRA 2012).
- Yamakawa, Namiki, Ishikawa, "Dynamic High-Speed Knotting of a Rope by a Manipulator" (IJARS 2013).
- Yamakawa, Namiki, Senoo, Ishikawa, "Dexterous Manipulation of a Rhythmic Gymnastics Ribbon..." (ICRA 2013).
- Chapter "Toward Dynamic Manipulation of Flexible Objects by High-Speed Robot System: From Static to Dynamic" (IntechOpen 2019).

**Sim vs real**: Real-world only. Planning uses simplified algebraic deformation models derived analytically; gravity often neglected under high-speed assumption.

**Learning method**: None — model-based motion planning + 1 kHz visual servoing. Included as the canonical classical reference for dynamic rope manipulation.

**Action representation**: Pre-planned, model-derived joint trajectories executed at 1 kHz with high-speed visual feedback corrections.

**Why cited in the surveys**: The foundational classical robotics line for dynamic high-speed rope manipulation — the prior generation of work that learning-based whip/casting methods are revisiting. Establishes the empirical fact that rope behavior can be controlled through carefully structured high-speed arm motions plus a compact, regime-specific deformation model. Reliance on 1 kHz vision + a specialized 4-DOF Barrett high-speed arm is a key practical caveat: their methods do not directly transfer to standard 7-DOF arms (Franka, UR5).

**Key result (if any)**: Demonstrated dynamic high-speed knotting and ribbon-shaping on a 4-DOF Barrett WAM with 1 kHz vision.
