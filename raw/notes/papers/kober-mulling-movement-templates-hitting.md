---
title: "Movement Templates for Learning of Hitting and Batting"
authors: "Jens Kober, Katharina Mülling, Oliver Kroemer, Christoph Lampert, Bernhard Schölkopf, Jan Peters"
venue: "ICRA 2010 / IROS 2010"
year: 2010
arxiv_id: null
doi: null
note_type: bibliography_only
sources: [report-1]
---

# Movement Templates for Learning of Hitting and Batting

**One-line gist**: DMP-based learning to hit a moving ball at a varying target with a required velocity at a specific time (table tennis / baseball forehand).

**Task setup**: Hitting a ball with a robot arm at a varying target, where the contact must occur with a required velocity at a specified time. Tasks include table tennis and baseball-style forehand strikes.

**Sim vs real**: Real-robot demonstrations with DMP-based motor primitives.

**Learning method**: Movement templates / Dynamic Movement Primitives (DMPs); the DMP target and velocity at hitting time are adapted to each new target.

**Action representation**: Parameterized DMP whose hitting-point and velocity meta-parameters are adjusted to new targets.

**Why cited in the surveys**: One of the canonical references for goal-conditioned DMP-based dynamic manipulation. Cited in Report 1 as foundational for the "DMP / motor-primitive" action-representation family that runs through the rope-tip-target literature (IRP, Edraki, Nah).

**Key result (if any)**: Successful hitting/batting on real robots using adapted movement templates.
