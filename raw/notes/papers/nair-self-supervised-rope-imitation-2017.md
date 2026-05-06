---
title: "Combining Self-Supervised Learning and Imitation for Vision-Based Rope Manipulation"
authors: "Ashvin Nair, Dian Chen, Pulkit Agrawal, Phillip Isola, Pieter Abbeel, Jitendra Malik, Sergey Levine"
venue: "ICRA 2017"
year: 2017
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Combining Self-Supervised Learning and Imitation for Vision-Based Rope Manipulation

**One-line gist**: Inverse-dynamics CNN trained on 60K self-supervised rope interactions; imitates a human demo by composing inverse-dynamics steps to drive a rope into a target image configuration.

**Task setup**: Quasi-static rope shaping into a target image (knot tying, S-shapes, etc.) using a Baxter dual-arm robot.

**Sim vs real**: Real-world only — 60K self-supervised pick-and-place interactions collected on hardware; no simulator.

**Learning method**: Self-supervised inverse-dynamics CNN. At deployment, a human demonstration provides a sequence of target images; the CNN proposes a pick-and-place action that transforms the current rope state toward the next demo image.

**Action representation**: Pick-and-place action (pick pixel + place pixel).

**Why cited in the surveys**: Foundational pre-cursor for image-conditioned rope manipulation. Cited as the canonical entry-point combining self-supervised data collection with imitation-style goal conditioning for DLOs. Quasi-static, image-conditioned — predates the dynamic-whip line.

**Key result (if any)**: Successful rope shaping into target image goals on a real Baxter from 60K self-supervised interactions.
