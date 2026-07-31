---
name: "Huazhe Xu"
affiliation: "Institute for Interdisciplinary Information Sciences (IIIS), Tsinghua University; Shanghai Qi Zhi Institute"
research_areas: [robot-learning, reinforcement-learning, visual-reinforcement-learning, dexterous-manipulation, dynamic-manipulation, generative-models-for-robotics]
scholar: "https://www.semanticscholar.org/author/2386620187"
date_updated: 2026-07-30
type:
  kind: researcher
---

## Research areas

Robot learning at Tsinghua IIIS and Shanghai Qi Zhi, spanning visual reinforcement learning, dexterous and dynamic manipulation, and generative (diffusion / flow-matching) policies for control. The thread relevant to this wiki is the group's push into *dynamic* manipulation — tasks where momentum and release timing decide the outcome and where the planned-vs-executed gap dominates — attacked with learned generative motion models rather than with hand-designed action parameterizations plus residual corrections.

## Recent work

- [[da-mmp-learning-coordinated-accurate-throwing]] — DA-MMP (arXiv 2509.23721, 2025), senior author: motion manifold primitives extended to variable-length trajectories, learned from 90k planner-generated throws, with a latent conditional flow-matching model trained on 60 executed trials labeled by measured landing position.

## My notes

Senior-author node for the closest published validation of the rope-swing project's base recipe (sweep → compact trajectory parameterization → conditional flow matching conditioned on achieved outcomes). Worth tracking as a group rather than as a single paper: the DA-MMP conclusion explicitly lists object-shape generalization, release orientation/spin control, and broader goal-conditioned dynamic tasks as next steps — the second of those is the direction axis the project claims as its own novelty, so a follow-up from this group is the most likely source of a scoop. The S2 author record is fragmented (10 papers, h-index 6 at ingest) and badly understates the actual output; do not use it for importance signals.
