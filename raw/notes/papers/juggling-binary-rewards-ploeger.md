---
title: "High Acceleration Reinforcement Learning for Real-World Juggling with Binary Rewards"
authors: "Kai Ploeger, Michael Lutter, Jan Peters"
venue: "CoRL 2020"
year: 2020
arxiv_id: "2010.13483"
doi: null
note_type: bibliography_only
sources: [report-2, l1c-5]
---

# High Acceleration RL for Real-World Juggling with Binary Rewards

**One-line gist**: Episodic policy-search RL with binary catch reward on a Barrett WAM; 56 minutes of real-world data → up to 33-minute / ~4500-catch continuous juggling.

**Task setup**: Two-ball juggling on a Barrett WAM 7-DoF high-acceleration cable-driven arm. Each catch is a high-speed dynamic strike.

**Sim vs real**: Real-world only — learning is done directly on hardware (no sim2real). Highlights safety / sample-efficiency of high-acceleration learning on real arms.

**Learning method**: Episodic policy-search RL with a binary per-rollout catch/no-catch reward. Policy initialized from a hand-designed parameterized motion and refined via a sample-efficient RL update (relative-entropy / contextual REPS-style).

**Action representation**: Compact parameterized trajectory — a small set of via-points / DMP-like motor primitive parameters in joint or task space — defining an entire juggling cycle.

**Why cited in the surveys**: Two transferable ideas: (i) compact motor-primitive action space for high-acceleration cyclic motions, learned with binary success rewards; (ii) safe initialization plus on-hardware refinement when sim2real of a high-accel cable-driven / compliant arm is unreliable — the same regime as whipping a rope where Cosserat-rod sim has known reality gaps.

**Key result (if any)**: Learns from 56 minutes of real interaction with binary rewards only; final policy juggles continuously up to 33 minutes / ~4500 catches.
