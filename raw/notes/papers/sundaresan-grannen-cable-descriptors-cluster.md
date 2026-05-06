---
title: "Sundaresan / Grannen Berkeley AUTOLAB Cable Descriptor Cluster"
authors: "Priya Sundaresan, Jennifer Grannen, et al. (Goldberg group, UC Berkeley)"
venue: "CoRL/ICRA/RSS 2020–2022 (multiple)"
year: 2020
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Sundaresan / Grannen Berkeley AUTOLAB Cable Descriptor Cluster

**One-line gist**: A long line of Berkeley AUTOLAB papers (CoRL/ICRA/RSS 2020–2022) on cable descriptors, dense object descriptors trained on synthetic depth, autonomous untangling — quasi-static but seminal for cable representations.

**Task setup**: Several quasi-static cable manipulation tasks: untangling, smoothing, knot inspection, autonomous routing. Common element: a learned per-pixel cable descriptor (often based on dense object descriptors) that supports keypoint correspondences, knot detection, or task-conditioned planning.

**Sim vs real**: Mixed — synthetic depth simulators (e.g. Blender) for descriptor pretraining, plus real-world fine-tuning and deployment.

**Learning method**: Self-supervised dense descriptor learning (in the Florence/Schmidt tradition) extended to cables; combined with learned classifiers and downstream task planners.

**Action representation**: Quasi-static pick-and-place primitives parameterized by descriptor-derived keypoints.

**Why cited in the surveys**: Seminal cable-representation cluster. Provides the perception substrate (dense descriptors, keypoints) on which later Berkeley dynamic-DLO work (Robots-of-the-Lost-Arc, Planar Robot Casting, Free-End Cable) builds.

**Key result (if any)**: Established dense descriptor and keypoint-based perception as the dominant cable representation in mid-2020s manipulation work.
