---
title: "Berkeley AUTOLAB Cable Untangling Series — HULK / LOKI / SGTM / SGTM 2.0 / SPiDERMan"
authors: "Ken Goldberg group (UC Berkeley AUTOLAB) — Sundaresan, Grannen, Viswanath, Lim, Wong, et al."
venue: "RSS 2021, ICRA 2021/2022, CoRL 2021/2022 (multiple)"
year: 2021
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Berkeley AUTOLAB Cable Untangling Cluster

**One-line gist**: Sequence of Berkeley AUTOLAB systems (HULK, LOKI, SGTM, SGTM 2.0, SPiDERMan) for untangling dense cable knots up to 3 m on a da Vinci or bimanual platform; primarily quasi-static, learned manipulation primitives over highly nonlinear cable configurations.

**Task setup**: Untangling dense knots in cables (up to 3 m) on a da Vinci or bimanual workstation. Quasi-static dominantly, but operates on extremely nonlinear cable configurations.

**Sim vs real**: Sim-to-real, using a Blender cable simulator and real bimanual / da Vinci hardware.

**Learning method**: Learned keypoint/perception detection + geometric planner (HULK); learned manipulation features + recovery policies in later iterations.

**Action representation**: Pick-and-place manipulation primitives.

**Why cited in the surveys**: The seminal cable-manipulation line of policies adjacent to dynamic methods. Cited as the lineage in which later dynamic Berkeley work (Robots-of-the-Lost-Arc, Planar Robot Casting, Free-End Cable) is rooted. The cluster shows how cable-manipulation primitives, learned descriptors, and recovery policies evolved.

**Key result (if any)**: Series of state-of-the-art untangling success rates on 3 m cables under occlusion-heavy configurations.
