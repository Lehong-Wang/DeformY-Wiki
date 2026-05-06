---
title: "Motor Control Beyond Reach — How Humans Hit a Target with a Whip"
authors: "Aleksei Krotov, Marta Russo, Moses Nah, Neville Hogan, Dagmar Sternad"
venue: "Royal Society Open Science 9(10):220581"
year: 2022
arxiv_id: null
doi: "10.1098/rsos.220581"
note_type: bibliography_only
sources: [report-1, report-2, l1d-1]
---

# Krotov et al. 2022 — Motor Control Beyond Reach: Hitting a Target with a Whip

**One-line gist**: Object-centered behavioural analysis of 16 humans hitting a 1.6 m bullwhip target at peak hand speed 4–6 m/s with peak tip speeds ~30 m/s (Mach 0.087) and per-block error 3–35 cm.

**Task setup**: 1.6 m bullwhip + 0.24 m rigid handle (336 g total) + horizontal spring on a 1.5 m tripod loaded with 5 kg. Distance personalized so fully extended arm + whip overlaps target by 5 cm. Target at participant shoulder height. 16 participants (7M/9F, 26.6 ± 5.0 yrs), both *discrete* and *rhythmic* hitting styles. 18 body markers + 10 custom soft-foam markers along the whip + 3 target markers; 12-camera Qualisys Oqus 3+ at 500 Hz.

**Sim vs real**: Human-subject behavioural experiment.

**Learning method**: None — object-centered behavioural analysis. Authors explicitly *reject* stochastic optimal feedback control formulations as not scaling to whip dynamics, and instead characterize whip evolution at peak hand speed as the "initial condition" that determines unfolding to target.

**Action representation**: N/A — descriptive analysis of human hand and whip kinematics.

**Why cited in the surveys**: The canonical empirical reference for what tip-velocity / whip-extension / azimuth a *successful* whip strike requires. Cited across all three reports as the human-motor-control parallel to robotics whip-targeting; key source for the Sternad/Hogan cluster on which Bai/Wang/Xiong robotics group builds.

**Key result (if any)**: Peak hand speed 6.1 ± 1.3 m/s discrete vs 4.5 ± 1.0 m/s rhythmic; peak tip speed 29.7 ± 7.8 m/s ≈ Mach 0.087; per-block error 3–35 cm; success up to 71%. Code/data: https://github.com/dondestamos/WhipTask_PerformanceWhipHand and https://doi.org/10.5281/zenodo.6987213.
