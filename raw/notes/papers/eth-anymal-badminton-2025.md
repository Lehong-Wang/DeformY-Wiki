---
title: "Learning Coordinated Badminton Skills for Legged Manipulators"
authors: "Yuntao Ma et al. (ETH RSL + RAI Institute)"
venue: "Science Robotics 2025"
year: 2025
arxiv_id: "2505.22974"
doi: null
note_type: bibliography_only
sources: [report-2]
---

# Learning Coordinated Badminton Skills for Legged Manipulators

**One-line gist**: ANYmal-D + DynaArm hits an incoming shuttlecock with a racket toward predicted intercept points; unified RL whole-body visuomotor policy + perception-noise model + shuttlecock predictor; constrained RL for safe high-velocity arm swings (up to 12.06 m/s).

**Task setup**: Whole-body badminton on ANYmal-D + DynaArm. Robot must intercept and hit incoming shuttlecocks toward predicted positions. Multi-rally play against humans.

**Sim vs real**: Sim-trained, real-deployed. Includes a learned shuttlecock predictor and perception-noise model.

**Learning method**: Unified RL whole-body visuomotor policy + shuttlecock predictor + constrained RL for safe high-velocity arm swings.

**Action representation**: Whole-body joint commands from the unified policy.

**Why cited in the surveys**: Cited in Report 2 as a recent striking/catching analog — high-velocity (up to 12 m/s) arm swings on a legged base, with constrained RL for safety. Methodologically related to whip-targeting because of the high tip-velocity regime and whole-body coordination.

**Key result (if any)**: Rallies up to 10 consecutive shots vs human; emergent gait adaptation by distance.
