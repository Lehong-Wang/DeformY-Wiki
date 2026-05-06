---
title: "Learning the Skill of Archery by a Humanoid Robot iCub"
authors: "Petar Kormushev, Sylvain Calinon, Darwin G. Caldwell"
venue: "IEEE Humanoids ~2010 / ICAR ~2010 (no arXiv)"
year: 2010
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-3]
---

# Learning the Skill of Archery by a Humanoid Robot iCub

**One-line gist**: 53-DOF iCub humanoid learns to shoot arrows so they hit the center of a target using visual feedback, comparing PoWER and the authors' ARCHER regression-based method; canonical low-dimensional target-conditioned action embedding.

**Task setup**: iCub humanoid (53 DOF) shoots arrows at a target with the goal of hitting the center, using a camera for visual feedback.

**Sim vs real**: Both — simulation comparisons and real-robot results.

**Learning method**: Comparison of Policy Learning by Weighting Exploration with the Returns (PoWER) and the authors' own ARCHER regression-based method.

**Action representation**: A 3D vector describing the relative position of the robot's two hands, controlling shot direction and velocity; the rest of the motion is handled by inverse kinematics.

**Why cited in the surveys**: Cited in Report 3 as a target-conditioned dynamic-manipulation analog that demonstrates a hard ballistic task can be learned via an extremely low-dimensional action embedding (3D hand-relative-position) rather than full trajectory learning. No arXiv source — bibliography note covers it from Report 3 description; flagged in the task brief as one of two papers without open-access source.

**Key result (if any)**: Successful arrow shooting at target center on iCub via the ARCHER method.
