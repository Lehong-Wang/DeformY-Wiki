---
title: "Deep Reinforcement Learning of Robotic Manipulation for Whip Targeting"
authors: "Bai, Wang, Xiong, Boukas (SDU / University of Southern Denmark)"
venue: "IEEE SIMPAR 2025 (DOI 10.1109/SIMPAR62925.2025.10979109)"
year: 2025
arxiv_id: null
doi: "10.1109/SIMPAR62925.2025.10979109"
note_type: bibliography_only
sources: [report-1, report-2, l1a-5]
---

# Deep Reinforcement Learning of Robotic Manipulation for Whip Targeting

**One-line gist**: SAC/PPO/TD3 deep RL on a 7-DoF robot arm holding a multi-segment whip in MuJoCo, hitting a 3D target in <1.5 s; pairs with OIAC for sim-to-real on real hardware.

**Task setup**: 7-DOF robot arm holding a multi-segment whip in MuJoCo must hit a 3D target in <1.5 s. Whip nodes modeled as 2-DOF rotational spring-damper systems. State is (Δx, Δy, Δz) between whip tip and target.

**Sim vs real**: Trains in MuJoCo; sim-to-real via Online Impedance Adaptation Control (OIAC) and visual feedback. Reports trajectory similarity ~89% between simulated and physical whips.

**Learning method**: Deep RL — compares Soft Actor-Critic (SAC), PPO, TD3, nonlinear optimization, and a genetic algorithm. SAC outperforms others in generalization across target locations; PPO most robust under sparse rewards. Reduces average learning trials to <20% of baselines and triples reward.

**Action representation**: Joint-space motion primitive (joint trajectory) parameterized to be tracked by an impedance controller.

**Why cited in the surveys**: A direct match for the 3D rope/whip-tip target hitting task, framed as a DRL problem on a 7-DoF arm with sim-to-real. The most recent (2024–2025) peer-reviewed work in this niche, paired with an open journal companion (Wang & Xiong 2024 Journal of Bionic Engineering 21:1761–1774).

**Key result (if any)**: SAC/PPO best; trains in <20% of baseline trials; sim-vs-real trajectory similarity ~89%; OIAC reduces tracking error 13–22% vs constant-impedance baseline.
