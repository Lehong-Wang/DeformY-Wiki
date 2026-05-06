# Literature Map: Goal-Conditioned Dynamic Rope-Tip Manipulation (Whip-to-3D-Target)

*Generated 2026-05-06 | Skill: `/research-scout --mode deep` | 10 subagents | 49 cached sources*

## Intent
- **Topic**: goal-conditioned dynamic manipulation of ropes / deformable linear objects with a robot arm (whip/swing the tip to a 3D target)
- **Question**: How is goal-conditioned dynamic rope-tip target-reaching being learned today, and what simulators / methodological analogs from goal-conditioned throwing apply?
- **Focus**: four orthogonal slices — (1) DLO dynamic whip-to-target with learned policy, (2) DLO dynamic manipulation with target conditioning, (3) goal-conditioned throwing/hitting of rigid objects as methodological analogs, (4) classical bullwhip control + Cosserat/DER simulators
- **Mode**: deep
- **Domain**: robotics (RL + dynamic manipulation + deformable objects)

## What's Happening Right Now

The field has a single working real-hardware recipe — a low-dimensional parameterized swing primitive (5–10 params, joint-space min-jerk or task-space waypoints) refined by either iterative residual on observed tip trajectories (IRP-style, Chi 2022) or one-shot supervised regression (PRC, Free-End Cable). 2025–2026 has produced exactly one published 3D-rope-whip-to-target benchmark with leaderboard numbers (Lan et al. arXiv 2505.17434, "DIDP" — diffusion policy + physics-informed test-time adaptation, sim-only, 84.3 % within 5 cm and 20.8 % within 1 cm), and the bleeding-edge 2026 arXiv shows three distinct extension axes accelerating in parallel: data-generation (NVIDIA SoftMimicGen 2603.25725 explicitly names dynamic whipping as a benchmark behavior), Dreamer-style world-modelling for DLOs (RopeDreamer 2604.28161 from the Jan Peters lab), and high-fidelity differentiable Cosserat-rod simulators wired to RL training stacks (DEFORM CoRL 2024 + DEFT 2025). Throwing analogs are converging on a "low-rate analytic prior + high-rate residual" architecture (TossingBot's scalar release-velocity residual now generalized in ETH 2025 IROS to 100 Hz nominal + 400 Hz residual whole-body throwing) that has not yet been transferred to whipping.

## Research Landscape

The whip-to-target slice is held by a small set of labs: **Columbia/TRI (Shuran Song, Cheng Chi)** for the foundational IRP line; **UC Berkeley AUTOLab (Goldberg, Ichnowski)** for cable casting and free-end cable dynamics; **Northwestern Polytechnical + Khalifa (Federico Renda's group)** for the DIDP 3D-whip benchmark and the underlying reduced-order Cosserat sim; **Northeastern + MIT (Sternad, Hogan, Ramezani)** for human-inspired classical whip control (Krotov, Nah, Edraki). Throwing analogs are dominated by **ETH RSL (Marco Hutter, Yuntao Ma)** with the IROS 2025 whole-body residual policy and Science Robotics 2025 ANYmal badminton, **Princeton/Google (Andy Zeng)** for TossingBot's living lineage, **EPFL LASA (Aude Billard)** for Throw-Flip impulse-momentum decoupling, and **Padova (Dalla Libera)** for MC-PILOT release-delay MBRL. DLO-simulator work concentrates at **UCLA SCI Lab (Jawed)** — the DisMech / Py-DiSMech / MAT-DiSMech family — and **U-Mich ROAHM (Vasudevan)** with the differentiable DEFORM/DEFT line. Primary venues: ICRA, IROS, RSS, CoRL, T-RO, RA-L, IJRR, plus dense arXiv cs.RO traffic. There is exactly one dedicated benchmark (DIDP 3D rope-whip) and one implicit benchmark (DaXBench WhipRope task in JAX). No Papers-with-Code leaderboard exists for either; reproductions are ad hoc.

## Active Research Threads

### Iterative-residual / parameterized swing primitive learning
Learn a **low-dimensional swing-primitive parameterization** and refine it iteratively at deployment using observed deviations.

#### Iterative Residual Policy: for Goal-Conditioned Dynamic Manipulation of Deformable Objects (Mar 2022)
- **Authors**: Cheng Chi, Benjamin Burchfiel, Eric Cousineau, Siyuan Feng, Shuran Song (Columbia / TRI)
- **Venue**: RSS 2022 (Best Paper) → IJRR 2024
- **What they do**: UR5 whips a rope to hit a 3D tip target; learns a delta-dynamics network over a parameterized open-loop swing trajectory in simulation, iteratively refines the action params from observed real trajectories at deployment time; transfers sim-only training to real UR5 and Sawyer with no real-rope finetuning.
- **Key result**: Generalizes zero-shot across multiple unseen ropes and across embodiments; RSS 2022 Best Paper.
- **Code**: https://github.com/columbia-ai-robotics/irp
- **Link**: https://arxiv.org/abs/2203.00663

#### Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy (DIDP) (May 2025)
- **Authors**: Guanzhou Lan, Yuqi Yang, Anup T. Mathew, Feiping Nie, Rong Wang, Xuelong Li, Federico Renda, Bin Zhao (NPU + Khalifa + TeleAI)
- **Venue**: arXiv 2505.17434
- **What they do**: First dedicated 3D-rope-whip benchmark on a 20-DoF reduced-order GVS Cosserat sim; proposes Dynamics-Informed Diffusion Policy (DIDP) — diffusion policy pretrained on inverse-dynamics imitation data, then physics-informed test-time adaptation enforcing kinematic + dynamics priors during denoising. Action = full reduced-order joint trajectory (no primitive).
- **Key result**: 93.9 / 84.3 / 62.3 / 20.8 % at 10 / 5 / 2 / 1 cm tip-to-goal; 3.6 cm mean tip error; outperforms diffusion-policy-only (88.4 / 80.0 / 61.6 / 19.0) and trajectory-opt-only (15.5 / 6.8 / 1.8 / 0.7); sim-only.
- **Link**: https://arxiv.org/abs/2505.17434

#### Implicit Physics-aware Policy for Dynamic Manipulation of Rigid Objects via Soft Body Tools (Feb 2025)
- **Authors**: Zixing Wang, Ahmed H. Qureshi (Purdue)
- **Venue**: ICRA 2025 / arXiv 2502.05696
- **What they do**: Goal-conditioned **one-shot** rope-as-tool transport of rigid objects to distant 3D targets; sim-train + real-deploy; implicit system identification encoder (no test-time refinement loop) + goal-conditioned action predictor outputting a single end-effector trajectory. Explicitly contrasts with IRP by replacing the per-trial residual loop with up-front implicit sysID.
- **Key result**: One-shot success on unseen rope physics.
- **Link**: https://arxiv.org/abs/2502.05696

#### Learning Deformable Object Manipulation Using Task-Level Iterative Learning Control ("flying knot") (Feb 2026)
- **Authors**: Suresh & Atkeson et al. (CMU Robotics Institute)
- **Venue**: arXiv 2602.21302
- **What they do**: Task-Level ILC for non-planar dynamic rope manipulation, demonstrated on the "flying knot" (arcing strike near rope end to flip it through a self-formed loop); learns directly on hardware (xArm 7) from a single human demo plus a simplified rope model; no large-scale sim.
- **Key result**: 100 % success in ≤ 10 trials across 7 rope/cable types; transfers across rope types in 2–5 trials.
- **Link**: https://arxiv.org/abs/2602.21302

#### ActivePusher: Active Learning and Planning with Residual Physics for Nonprehensile Manipulation (Jun 2025)
- **Authors**: Zhuoyun Zhong, Seyedali Golestaneh, Constantinos Chamzas (WPI ELPIS Lab)
- **Venue**: ICRA 2026
- **What they do**: Residual-physics dynamics + Neural Tangent Kernel uncertainty + BAIT active-learning to pick informative skill parameters; integrated with kinodynamic planner that biases sampling toward low-uncertainty actions. Skill-parameter action representation. Methodological analog for sample-efficient residual-physics learning.
- **Key result**: Higher prediction accuracy and planning success with fewer samples vs. baselines.
- **Code**: https://github.com/elpis-lab/ActivePusher
- **Link**: https://arxiv.org/abs/2506.04646

### One-shot / single-step regression for DLO tip targeting
Predict a single open-loop trajectory or a low-D plan parameterization from the (image, goal) pair, deploy without iteration.

#### Robots of the Lost Arc: Self-Supervised Learning to Dynamically Manipulate Fixed-Endpoint Cables (Nov 2020 / ICRA 2022)
- **Authors**: Harry Zhang, Jeffrey Ichnowski, Daniel Brown, Ramtin Hosseini, Ken Goldberg (UC Berkeley AUTOLab)
- **Venue**: ICRA 2022
- **What they do**: UR5 high-speed swings a fixed-endpoint cable to vault, knock, or weave; CNN predicts a 3D **apex point** parameterizing an arcing minimum-jerk trajectory solved via QP under joint limits — "apex" is now the canonical low-D plan parameterization.
- **Key result**: Outperforms human-chosen apex baselines on 5 cables across vaulting / knocking / weaving.
- **Link**: https://arxiv.org/abs/2011.04840

#### Planar Robot Casting with Real2Sim2Real Self-Supervised Learning (Nov 2021 / ICRA 2022)
- **Authors**: Vincent Lim, Huang Huang, Lawrence Yunliang Chen, Jonathan Wang, Jeffrey Ichnowski, Daniel Seita, Michael Laskey, Ken Goldberg (UC Berkeley AUTOLab)
- **Venue**: ICRA 2022
- **What they do**: Single planar end-effector trajectory slides the other end of a cable to a 2D target beyond the workspace; Real2Sim2Real auto-tunes Isaac Gym/PyBullet via Differential Evolution, then trains single-step action policies (GP and NN) from mixed sim+real data.
- **Key result**: Per-cable R2S2R median tip error 8 % / 12 % / 14 % of cable length over 240 trials; analytic Cast-and-Pull baseline 61 % on the hardest cable.
- **Link**: https://arxiv.org/abs/2111.04814

#### Self-Supervised Learning of Dynamic Planar Manipulation of Free-End Cables (May 2024)
- **Authors**: Jonathan Wang, Huang Huang, Vincent Lim, Harry Zhang, Jeffrey Ichnowski, Daniel Seita, Yunliang Chen, Ken Goldberg (UC Berkeley AUTOLab)
- **Venue**: arXiv 2405.09581
- **What they do**: Single-step dynamic robot motion to land the **free tip** of a cable at a 2D target potentially outside the workspace; PyBullet differentially tuned to physical cables; supervised regression of trajectory-to-tip-target. Direct extension of Planar Robot Casting from fixed-end to free-end.
- **Key result**: Median tip error 22 / 24 / 34 % of cable length on 3 cables; +21 % over analytic Polar-Casting, +7 % over GP.
- **Link**: https://arxiv.org/abs/2405.09581

#### Goal-Image Conditioned Dynamic Cable Manipulation through Bayesian Inference and Multi-Objective Black-Box Optimization (Jan 2023)
- **Authors**: Kuniyuki Takahashi, Tadahiro Taniguchi (Preferred Networks)
- **Venue**: ICRA 2023
- **What they do**: Stochastic forward model for dynamic cable manipulation; **target encoded as a goal image**; action = joint-angle trajectory of 3-DoF arm; black-box optimization (TPE) handles non-differentiable objectives; real robot.
- **Key result**: Realizes goal-image-specified cable configurations on real 3-DoF arm.
- **Link**: https://arxiv.org/abs/2301.11538

### End-to-end RL on dynamic / quasi-dynamic DLOs with explicit goal conditioning
Train a policy network end-to-end with the target as direct input; vary action space and target encoding.

#### DexDLO: Learning Goal-Conditioned Dexterous Policy for Dynamic Manipulation of Deformable Linear Objects (Dec 2023 / ICRA 2024)
- **Authors**: Sun Zhaole, Jihong Zhu, Robert B. Fisher (Edinburgh / York)
- **Venue**: ICRA 2024
- **What they do**: Model-free goal-conditioned RL on a fixed-base dexterous hand for DLO grabbing/pulling/end-tip placement; policy conditioned on a 3D goal point G and a tracked DLO point X; trained MuJoCo, sim-only; action = hand joint angles.
- **Key result**: 5 unified DLO tasks learned with the same hyperparameters in MuJoCo.
- **Link**: https://arxiv.org/abs/2312.15204

#### Deep Reinforcement Learning of Robotic Manipulation for Whip Targeting (2025)
- **Authors**: SDU (Univ. of Southern Denmark) team
- **Venue**: IEEE SIMPAR 2025 (companion: Journal of Bionic Engineering 21:1761–1774, 2024)
- **What they do**: 7-DoF arm holding a whip must hit a 3D target in <1.5 s; MuJoCo whip-target environment; compares SAC, PPO, and others on joint-space motion plans; pairs DRL with Online Impedance Adaptation Control (OIAC) for tracking and visual-feedback sim2real.
- **Key result**: SAC/PPO best; trains in <20 % of baseline trials; sim-vs-real trajectory similarity ~89 %; tracking error reduced 13–22 % vs constant-impedance baseline.
- **Link**: https://link.springer.com/article/10.1007/s42235-024-00534-2

#### Offline Goal-Conditioned Reinforcement Learning for Shape Control of Deformable Linear Objects (Mar 2024)
- **Authors**: Rita Laezza, Mohammadreza Shetab-Bushehri, Gabriel Arslan Waltersson, Erol Özgür, Youcef Mezouar, Yiannis Karayiannidis (Chalmers / KTH / Inst. Pascal)
- **Venue**: arXiv 2403.10290
- **What they do**: Planar DLO shape control as goal-conditioned offline RL with TD3+BC + HER-style data augmentation; target = discretized planar polyline; quasi-static EE-pose action; deployed on real robot for soft rope and elastic cord.
- **Key result**: Beats shape-servoing baseline on curvature-inversion task.
- **Link**: https://arxiv.org/abs/2403.10290

#### Self-Curriculum Model-based Reinforcement Learning for Shape Control of Deformable Linear Objects (Feb 2026)
- **Authors**: Zhaowei Liang et al. (Tsinghua)
- **Venue**: arXiv 2602.21816
- **What they do**: Two-stage shape-control: model-based RL with ensemble dynamics + self-curriculum HER-style goal generation in the large-deformation regime, then Jacobian visual servoing for fine convergence; targets are full DLO shapes (image / keypoint polyline); EE-pose action; trained sim, zero-shot real.
- **Key result**: 30 / 30 success on real DLOs of varied size and material (zero-shot sim-to-real).
- **Link**: https://arxiv.org/abs/2602.21816

#### Hierarchical DLO Routing with Reinforcement Learning and In-Context Vision-language Models (Oct 2025)
- **Authors**: Mingen Li, Houjian Yu, Yixuan Huang, Youngjin Hong, Hantao Ye, Changhyun Choi (UMN)
- **Venue**: arXiv 2510.19268 (ICRA 2026 candidate)
- **What they do**: Hierarchical autonomous routing of cables through clip sequences; VLM does in-context high-level planning into language sub-goals; RL low-level skills + failure-recovery sub-policy execute. Action = motion primitives; observation = vision.
- **Key result**: 92.5 % overall success on long-horizon routing; ~50 percentage-points over the next baseline.
- **Link**: https://arxiv.org/abs/2510.19268

#### Generalizable whole-body global manipulation of deformable linear objects by dual-arm robot in 3-D constrained environments (Oct 2023 / IJRR 2025)
- **Authors**: Mingrui Yu, Kangchen Lv, Changhao Wang, Yongpeng Jiang, Masayoshi Tomizuka, Xiang Li (Tsinghua / UC Berkeley)
- **Venue**: IJRR 2025
- **What they do**: Global planning + closed-loop manipulation for moving and shaping DLOs in 3D constrained environments with dual-arm robots; simplified DLO energy model + adaptive online DLO motion model; goal = configuration in a constrained env.
- **Key result**: Generalizable across various DLOs and constrained 3D scenes; real-robot validated.
- **Link**: https://arxiv.org/abs/2310.09899

### Goal-conditioned throwing as methodological analog
Project rigid-object goal-conditioned throwing techniques (action representations, sim2real, target encodings) onto the rope-tip target setting.

#### TossingBot: Learning to Throw Arbitrary Objects with Residual Physics (Mar 2019 / T-RO 2020)
- **Authors**: Andy Zeng, Shuran Song, Johnny Lee, Alberto Rodriguez, Thomas Funkhouser (Princeton / Google / Columbia / MIT)
- **Venue**: RSS 2019 Best Systems → IEEE T-RO 2020
- **What they do**: UR5 grasps from bin and throws into 12 boxes outside reach; real-world self-supervised; FCN over RGB-D heightmap predicts grasp + **scalar residual on tangential release velocity** on top of an analytic ballistic prior; release angle fixed at 45°. Goal encoded by broadcasting `v̂` as a 128-channel feature image.
- **Key result**: 84.7 % throw accuracy on seen objects, 82.3 % on unseen, 83.9 % on unseen target locations (real); 514 MPPH throughput.
- **Link**: https://arxiv.org/abs/1903.11239

#### Data-efficient Robotic Object Throwing with Model-Based RL (MC-PILOT) (Feb 2025)
- **Authors**: Niccolò Turcato, Giulio Giacomuzzo, Matteo Terreran, Davide Allegro, Ruggero Carli, Alberto Dalla Libera (U. Padova)
- **Venue**: arXiv 2502.05595 (extension of ICRA 2023 conf paper)
- **What they do**: Pick-and-throw to user 3D target on real Franka Panda; PILCO-style probabilistic MBRL with **explicit release-delay model** in the loop; action = release-state vector (release pos + 3-D release velocity).
- **Key result**: Outperforms model-free RL + analytic baselines, generalizes to unseen targets with few rollouts.
- **Link**: https://arxiv.org/abs/2502.05595

#### Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration (Jun 2025)
- **Authors**: Yuntao Ma, Yang Liu, Kaixian Qu, Marco Hutter (ETH Zurich RSL)
- **Venue**: IROS 2025
- **What they do**: ANYmal-D + DynaArm whole-body throw to 3–6 m 3D targets; sim-trained, real-deployed; 100 Hz nominal MPC tracker + 400 Hz RL residual policy + pullback-tube acceleration optimizer; action = end-effector velocity reference + joint-level residuals (continuous, not scalar).
- **Key result**: 0.276 m mean landing error at 6 m; 56.8 % hit rate vs 15.2 % for humans on a 25×28 cm target at 3–4 m; legs add 53.4 % of angular impulse.
- **Link**: https://arxiv.org/abs/2506.16986

#### Learning to Throw-Flip (Oct 2025)
- **Authors**: Yang Liu, Bruno Da Costa, Aude Billard (EPFL LASA)
- **Venue**: arXiv 2510.10357
- **What they do**: Real revolute arm throws an object to a target 6-DoF **pose**; impulse-momentum motion design decouples linear/angular release velocity, then regression learns a residual on the free-flight model; action = release linear vel + release angular vel.
- **Key result**: ±5 cm, ±45° in dozens of trials; 40 % sample reduction vs end-to-end; 70 % transfer speedup with COM-shifted objects.
- **Link**: https://arxiv.org/abs/2510.10357

#### High Acceleration RL for Real-World Juggling with Binary Rewards (Oct 2020)
- **Authors**: Kai Ploeger, Michael Lutter, Jan Peters (TU Darmstadt IAS / MPI-IS)
- **Venue**: CoRL 2020
- **What they do**: Two-ball juggling on Barrett WAM; real-world only; episodic policy search with binary catch reward; action = compact parameterized motor-primitive trajectory params (DMP-like via-points).
- **Key result**: 56 min of real-world data → up to 33 min / ~4500 continuous catches.
- **Link**: https://arxiv.org/abs/2010.13483

#### Reinforcement Learning to Adjust Parametrized Motor Primitives to New Situations (2012)
- **Authors**: Jens Kober, Andreas Wilhelm, Erhan Oztop, Jan Peters
- **Venue**: Autonomous Robots / IJCAI 2011
- **What they do**: Robot dart-throwing + ball-paddling; meta-parameter RL (cost-regularized kernel regression / reward-weighted regression) adjusts **DMP global parameters** (release angle + release velocity) to new dartboard targets; real Barrett WAM / BioRob / CBi / Kuka KR6.
- **Key result**: Hits all dartboard target positions reliably from 260 rollouts.
- **Link**: http://www.jenskober.de/publications/Kober2012Auro.pdf

#### Whole-Body Dynamic Throwing with Legged Manipulators (Oct 2024)
- **Authors**: ETH RSL group (lead-author overlap with the IROS 2025 paper)
- **Venue**: arXiv 2410.05681
- **What they do**: Humanoid + ANYmal-C+Kinova quadruped throw to 3D target; sim (IsaacGym, PPO) with adaptive curriculum + sparse reward; whole-body joint-velocity policy; sim2real to humanoid.
- **Key result**: 73 cm error within 5 m radius; full-body throw outperforms arm-only on range/accuracy/stability.
- **Link**: https://arxiv.org/abs/2410.05681

#### Dynamic Throwing with Robotic Material Handling Machines (May 2024)
- **Authors**: ETH RSL group
- **Venue**: arXiv 2405.19001 (T-RO submission)
- **What they do**: 12-ton hydraulic excavator with passive-joint gripper learns dynamic throwing of objects to targets outside static reach; sim2real RL controller exploits passive-joint dynamics; action = joint-velocity trajectory.
- **Key result**: Throws individual objects accurately to targets outside the static reach zone.
- **Link**: https://arxiv.org/abs/2405.19001

#### Learning Coordinated Badminton Skills for Legged Manipulators (May 2025)
- **Authors**: Yuntao Ma et al. (ETH RSL + RAI Institute)
- **Venue**: Science Robotics 2025
- **What they do**: ANYmal-D + DynaArm hits incoming shuttlecock with racket toward predicted intercept; unified RL whole-body visuomotor policy + perception-noise model + shuttlecock predictor; constrained RL for safe high-velocity arm swings (up to 12.06 m/s); sim2real.
- **Key result**: Rallies up to 10 consecutive shots vs human; emergent gait adaptation by distance.
- **Link**: https://arxiv.org/abs/2505.22974

#### DartBot: Overhand Throwing of Deformable Objects with Tactile Sensing and RL (2025)
- **Authors**: Shoaib Aslam, Pokuang Zhou, Krish Kumar, Hongyu Yu, Michael Wang, Yu She (Purdue / HKUST)
- **Venue**: IEEE T-ASE 2025
- **What they do**: Robot arm + high-res tactile fingertip throws non-rigid darts/small deformables to a landing target; trained directly on hardware (no sim2real); two pre-throw tilting actions encode tactile features; RL policy maps (tactile embedding, target) → throw parameters.
- **Key result**: 95 % success on unseen objects; 3.15 cm mean target error.
- **Link**: https://krishkumar1.github.io/dartbot-website/

### Classical whip-tip control: bullwhip physics + model-based primitive optimization
Analytical / numerical models of the whip and derivative-free optimization over a low-D handle trajectory.

#### Shape of a Cracking Whip (2002)
- **Authors**: Alain Goriely, Tyler McMillen (Univ. of Arizona)
- **Venue**: Physical Review Letters 88(24):244301
- **What they do**: Analytical + numerical model of an inextensible inhomogeneous planar elastic rod with fixed-free BCs; decomposes whip-tip acceleration into tension, taper, and boundary-condition contributions.
- **Key result**: Taper alone gives ~10× wave-speed amplification, free end adds another ~2–3× — predicted total breaks Mach 1.
- **Link**: https://journals.aps.org/prl/abstract/10.1103/PhysRevLett.88.244301

#### The puzzle of whip cracking — uncovered by a correlation of whip-tip kinematics with shock wave emission (1998)
- **Authors**: Peter Krehl, Stephan Engemann, Dieter Schwenkel (Ernst-Mach-Institut)
- **Venue**: Shock Waves 8(1):1–9
- **What they do**: 9 kHz shadowgraph high-speed photography of a whip cracker correlated with shock-wave emission.
- **Key result**: Peak tip Mach 2.19, ~50,000 g acceleration, supersonic phase ~1.2 ms; sonic boom emitted at the turn-around point (M ≈ 2), not at M = 1.
- **Link**: https://link.springer.com/article/10.1007/s001930050093

#### Motor control beyond reach — how humans hit a target with a whip (2022)
- **Authors**: Aleksei Krotov, Marta Russo, Moses Nah, Neville Hogan, Dagmar Sternad (Northeastern + MIT)
- **Venue**: Royal Society Open Science 9(10):220581
- **What they do**: Object-centered behavioural analysis of 16 humans hitting a target with a 1.6 m bullwhip; measures hand and whip kinematics at peak hand speed and links them to error.
- **Key result**: Peak hand speed 6.1 ± 1.3 m/s discrete vs 4.5 ± 1.0 m/s rhythmic; peak tip speed 29.7 m/s ≈ Mach 0.087; per-block error 3–35 cm, success up to 71 %.
- **Code**: https://github.com/dondestamos/WhipTask_PerformanceWhipHand
- **Link**: https://royalsocietypublishing.org/doi/10.1098/rsos.220581

#### Learning to manipulate a whip with simple primitive actions — a simulation study (2023)
- **Authors**: Moses C. Nah, Aleksei Krotov, Marta Russo, Dagmar Sternad, Neville Hogan
- **Venue**: iScience 26(8):107395
- **What they do**: 4-DoF arm + 50-DoF segmented whip in MuJoCo; classical black-box trajectory optimization (DIRECT-L from NLopt) over a **9-parameter minimum-jerk joint-space primitive**; goal = 3D target.
- **Key result**: 39–249 iterations to converge; single discrete submovement reaches all 6/6 targets (3 within reach, 3 at max reach), 3 cm-radius success sphere.
- **Code**: https://github.com/MosesAndLily/whip-project-targeting
- **Link**: https://www.cell.com/iscience/fulltext/S2589-0042(23)01472-4

#### Human-inspired control of a whip: preparatory movements improve hitting a target (2024)
- **Authors**: Mahdiar Edraki, Rajiv Lokesh, Aleksei Krotov, Alireza Ramezani, Dagmar Sternad (Northeastern)
- **Venue**: IEEE BioRob 2024 (also ICRA 2025 RMDO spotlight)
- **What they do**: Forward-dynamics Lagrangian model of a 25-link 1 m whip + PD joint stiffness/damping; MATLAB pattern-search over 5-param "striking-only" vs 9+1-param "preparing-and-striking" minimum-jerk handle trajectories.
- **Key result**: Model validation < 5 cm tip RMS vs physical 3D-printed whip; preparing-and-striking strategy reaches far targets at 1.2 m unreachable by striking-only; 1 cm hit threshold.
- **Link**: https://pmc.ncbi.nlm.nih.gov/articles/PMC11715529/

#### Casting and Winding Manipulation with Hyper-Flexible Manipulator (2002 / 2006)
- **Authors**: Hiromi Mochiyama, Takahiro Suzuki, Yasuhiro Ebihara
- **Venue**: ICRA 2002 / IROS 2006
- **What they do**: Hyper-flexible manipulator modeled as underactuated multi-link with passive joints; explicit casting motion sends the tip of a flexible cable to a 3D target via inverse-dynamics planning of a single base actuator.
- **Key result**: Demonstrated casting + winding around remote targets; sub-tip accuracy demonstrated experimentally.
- **Link**: https://www.semanticscholar.org/paper/Casting-and-Winding-Manipulation-with-Manipulator-Suzuki-Ebihara/374f7ca9803f00162b49d17f69d828b837b81761

#### Whip waves (2003)
- **Authors**: Tyler McMillen, Alain Goriely
- **Venue**: Physica D 184:192–225
- **What they do**: Long-form asymptotic analysis of wave propagation along a tapered inextensible-unshearable inhomogeneous elastic rod; characterizes shock-wave geometry from the supersonic material point.
- **Key result**: Wave speed ∝ ρ(s)^(-1/4); explains the supersonic regime quantitatively.
- **Link**: https://goriely.com/publications/j29

### DLO simulators & differentiable physics for goal-conditioned training
Cosserat-rod / DER simulators wired (or wireable) to RL training stacks; differentiable variants enable gradient-based policy training.

#### DEFORM: Differentiable Discrete Elastic Rods for Real-Time Modeling of Deformable Linear Objects (Jun 2024)
- **Authors**: Yizhou Chen, Yiting Zhang, Zachary Brei, Tiancheng Zhang, Yuzhen Chen, Julie Wu, Ram Vasudevan (U-Mich ROAHM Lab)
- **Venue**: CoRL 2024 (PMLR v270)
- **What they do**: Reformulates Bergou-style DER as Differentiable DER (DDER) with analytical gradients of state w.r.t. parameters; combined with a learned residual head; PyTorch.
- **Key result**: Real-time DLO tracking + planning on two industrial arms (specific FPS not in abstract).
- **Code**: https://github.com/roahmlab/DEFORM
- **Link**: https://arxiv.org/abs/2406.05931

#### DEFT: Differentiable Branched Discrete Elastic Rods for Modeling Furcated DLOs in Real-Time (Feb 2025)
- **Authors**: Yizhou Chen et al. (U-Mich ROAHM Lab)
- **Venue**: arXiv 2502.15037
- **What they do**: Extends DEFORM/DDER to branched DLOs (wiring harnesses); models junction-point dynamics and mid-rope grasping.
- **Key result**: Real-time inference, planning demos on real branched cables.
- **Code**: https://github.com/roahmlab/DEFT
- **Link**: https://arxiv.org/abs/2502.15037

#### DaXBench: Benchmarking Deformable Object Manipulation with Differentiable Physics (Oct 2022 / ICLR 2023)
- **Authors**: Siwei Chen, Yiqing Xu, Cunjun Yu, Linfeng Li, Xiao Ma, Zhongwen Xu, David Hsu (NUS AdaComp)
- **Venue**: ICLR 2023 (Oral)
- **What they do**: JAX differentiable benchmark for rope/cloth/liquid/elastoplastic; rope = mass-spring + bending; **WhipRope is a built-in task**; OpenAI-Gym API; batched env support; APG / SHAC / PPO / CEM-MPC / Diff-MPC / ILD baselines.
- **Key result**: WhipRope baselines: APG 0.83 ± 0.01 (best), SHAC 0.66, PPO 0.25, ILD 0.70, CEM-MPC 0.34, Expert 1.00. Real-robot Push-Rope Sim2Real validated on Kinova-3 (quasi-static only).
- **Code**: https://github.com/AdaCompNUS/DaXBench
- **Link**: https://arxiv.org/abs/2210.13066

#### Accurate Simulation and Parameter Identification of DLOs using Discrete Elastic Rods in Generalized Coordinates (IROS 2025)
- **Authors**: Qi Jing Chen, Timothy Bretl, Quang-Cuong Pham (NTU + UIUC)
- **Venue**: IEEE/RSJ IROS 2025
- **What they do**: Integrates Bergou DER bending+twisting energies into MuJoCo's generalized-coordinate solver via force-lever analysis; adds parameter-ID pipeline fitting bending/twisting stiffness from real DLOs; not differentiable but MJX-compatible (CPU + JAX/GPU batched via MJX).
- **Key result**: Improved accuracy over MuJoCo native cable plugin; validated on three real DLOs in static / quasi-dynamic regimes.
- **Link**: https://arxiv.org/abs/2310.00911

#### DisMech: A Discrete Differential Geometry-based Physical Simulator for Soft Robots and Structures (Nov 2023 / RA-L 2024)
- **Authors**: Andrew Choi, Ran Jing, Andrew Sabelhaus, Mohammad Khalid Jawed (UCLA SCI Lab + BU + CMU)
- **Venue**: IEEE RA-L 2024
- **What they do**: Bergou-DER rods + discrete elastic shells; fully implicit Newton-solve time integration; C++ + Python + MATLAB; gradients via implicit solver enable inverse design / traj-opt.
- **Key result**: ~10× speed-up over prior DER simulators while keeping physical accuracy; validated against soft-robot hardware.
- **Code**: https://github.com/StructuresComp/dismech-rods
- **Link**: https://arxiv.org/abs/2311.18126

#### Discrete Elastic Rods (2008)
- **Authors**: Miklós Bergou, Max Wardetzky, Stephen Robinson, Basile Audoly, Eitan Grinspun (Columbia / Göttingen)
- **Venue**: ACM SIGGRAPH (TOG 27(3))
- **What they do**: Foundational DER formulation (Bishop-frame angle as material-frame DoF, quasi-static frame, dynamic centerline); the basis for DEFORM / DEFT / DisMech and ~all modern rope-simulation work in robotics.
- **Key result**: Validated buckling/stability/coupled-mode + knot-tying.
- **Link**: https://www.cs.columbia.edu/cg/rods/

#### CoRdE: Cosserat Rod Elements for the Dynamic Simulation of One-Dimensional Elastic Objects (2007)
- **Authors**: Jonas Spillmann, Matthias Teschner (Univ. Freiburg)
- **Venue**: SCA 2007
- **What they do**: Foundational FEM Cosserat rod simulator with continuous bending/twisting/stretching energies; CPU; not differentiable; reference physics for many later DLO sim2real works.
- **Key result**: Stable elastic-rod dynamics with self-contact; standard reference.
- **Link**: https://cg.informatik.uni-freiburg.de/publications/2007_SCA_corde.pdf

### Bleeding-edge 2026 sim-to-real adaptation for DLO policies
Jan–Apr 2026 arXiv preprints: data generation, world models, distributional and particle-embedding adaptation for DLO sim2real.

#### SoftMimicGen: A Data Generation System for Scalable Robot Learning in Deformable Object Manipulation (Mar 2026)
- **Authors**: Masoud Moghani, Mahdi Azizian, Animesh Garg, Yuke Zhu, Sean Huver, Ajay Mandlekar (NVIDIA / U-Toronto / UT-Austin)
- **Venue**: arXiv 2603.25725
- **What they do**: MimicGen-style automated data generation pipeline for deformable manipulation; sim-only suite **explicitly includes dynamic whipping of rope** plus threading and folding across single-arm, bimanual, humanoid, and surgical embodiments; trains imitation policies on synthesized data.
- **Key result**: First scaled data-gen system to enumerate dynamic whipping as a benchmark behavior in 2026 (specific whip-target numbers not disclosed in abstract).
- **Link**: https://arxiv.org/abs/2603.25725

#### RopeDreamer: A Kinematic Recurrent State Space Model for Dynamics of Flexible Deformable Linear Objects (Apr 2026)
- **Authors**: Tim Missal, Lucas Domingues, Berk Guler, Simon Manschitz, Jan Peters, Paula Dornhofer Paro Costa (TU Darmstadt + UNICAMP + Honda Research Institute)
- **Venue**: arXiv 2604.28161
- **What they do**: Dreamer-style RSSM forward-dynamics model for DLOs using a quaternionic kinematic-chain latent that preserves link-length constancy by construction; dual-decoder reconstruction + prediction.
- **Key result**: 40.52 % rollout-error reduction over 50-step rollouts; 31.17 % faster inference vs baseline.
- **Link**: https://arxiv.org/abs/2604.28161

#### A Distributional Treatment of Real2Sim2Real for Object-Centric Agent Adaptation in Vision-Driven Deformable Linear Object Manipulation (Mar 2026)
- **Authors**: Georgios Kamaras, Subramanian Ramamoorthy (Edinburgh)
- **Venue**: arXiv 2502.18615 v4 / IEEE RA-L 10(8) 2025
- **What they do**: DLO reaching task; likelihood-free inference computes posteriors over rope physical parameters, then drives domain-randomized model-free RL for object-specific visuomotor policies.
- **Key result**: Zero-shot real deployment; distinguishes parameterized DLOs from dynamic trajectories.
- **Link**: https://arxiv.org/abs/2502.18615

#### Rapid Adaptation of Particle Dynamics for Generalized Deformable Object Mobile Manipulation (RAPiD) (Mar 2026)
- **Authors**: Bohan Wu, Roberto Martín-Martín (UT-Austin), Li Fei-Fei (Stanford)
- **Venue**: arXiv 2603.18246
- **What they do**: 22-DoF mobile manipulator; RMA-style two-phase: privileged particle-position+mass embedding teacher in sim, then non-privileged visuomotor student inferring the embedding from RGB + actions for sim-to-real.
- **Key result**: > 80 % success on 2 real-world deformable mobile-manip tasks across varied dynamics and instances.
- **Link**: https://arxiv.org/abs/2603.18246

#### Real-to-Sim Robot Policy Evaluation with Gaussian Splatting Simulation of Soft-Body Interactions (2026)
- **Authors**: Kaifeng Zhang, Shuo Sha, Hanxiao Jiang, Changxi Zheng, Yunzhu Li (Columbia + SceniX + Google DeepMind)
- **Venue**: ICRA 2026
- **What they do**: Real-demo-trained imitation policies (diffusion-policy class) evaluated in Gaussian-splatting + soft-body physics replicas of real scenes; tasks include rope routing, plush-toy packing, T-block pushing; EE-trajectory action.
- **Key result**: Faithful policy ranking matching real-world rollouts.
- **Link**: https://real2sim-eval.github.io/

#### Coordinated Manipulation of Hybrid Deformable-Rigid Objects in Constrained Environments (Mar 2026)
- **Authors**: Anees Peringal, Anup Teejo Mathew, Panagiotis Iiatsis, Federico Renda (Khalifa Univ / KU-CARS)
- **Venue**: arXiv 2603.12940
- **What they do**: Quasi-static optimization (not learning) for hybrid deformable-rigid linear objects via strain-based Cosserat rod with analytical gradients; dual-arm hardware with 3-link hDLO.
- **Key result**: ~3 cm avg deformation error (5 % of link length); 33× speedup over finite-diff baselines.
- **Link**: https://arxiv.org/abs/2603.12940

## Current SOTA Snapshot

- **3D rope-whip benchmark (Lan et al. 2025, arXiv 2505.17434)** — DIDP at 93.9 / 84.3 / 62.3 / 20.8 % within 10 / 5 / 2 / 1 cm; 3.6 cm mean tip error; sim-only; baselines: Diffusion-Policy 88.4 / 80.0 / 61.6 / 19.0, Trajectory-Opt 15.5 / 6.8 / 1.8 / 0.7, Kinematics-only 77.3 / 69.9 / 43.4 / 10.7. (https://arxiv.org/abs/2505.17434)
- **DaXBench WhipRope (ICLR 2023, JAX)** — APG 0.83 ± 0.01 best, SHAC 0.66, ILD 0.70, PPO 0.25, CEM-MPC 0.34, Expert 1.00; only APG training script publicly released as of 2026-Apr. (https://github.com/AdaCompNUS/DaXBench)
- **Planar Robot Casting Real2Sim2Real (ICRA 2022)** — per-cable median tip-error 8 / 12 / 14 % of cable length; analytic Cast-and-Pull baseline 61 % on the hardest cable. (https://arxiv.org/abs/2111.04814)
- **Free-End Cable (arXiv 2024)** — 22 / 24 / 34 % per-cable median tip-error / cable length; analytic Polar-Casting 43 %, GP 29 %. (https://arxiv.org/abs/2405.09581)
- **TossingBot (T-RO 2020)** — 84.7 % seen-objects, 82.3 % unseen-objects, 83.9 % unseen-target-locations real-world; 514 MPPH. (https://arxiv.org/abs/1903.11239)
- **ETH Whole-Body Throwing (IROS 2025)** — 0.276 m mean error at 6 m, 0.429 m at 4 m; 56.8 % vs 15.2 % human hit rate on a 25 × 28 cm target; pullback-tube cuts max-error from 96.8 cm to 31.1 cm. (https://arxiv.org/abs/2506.16986)
- **ILC flying-knot (CMU 2026)** — 100 % success in ≤ 10 trials across 7 rope/cable types; 2–5 trials cross-rope transfer. (https://arxiv.org/abs/2602.21302)

## Active Code & Repos

- **columbia-ai-robotics/irp** — Iterative Residual Policy code, sim training, UR5 deployment for rope-whip-to-target and cloth-swing | ⭐~70 | Updated: 2023-06 | https://github.com/columbia-ai-robotics/irp
- **AdaCompNUS/DaXBench** — JAX differentiable rope/cloth/liquid benchmark with WhipRope task; APG training script released, others paper-only | ⭐~70 | Updated: 2023-05 | https://github.com/AdaCompNUS/DaXBench
- **roahmlab/DEFORM** — Differentiable DER PyTorch, real-DLO tracking + planning, CoRL 2024 code | ⭐50–100 | Updated: 2025-03 | https://github.com/roahmlab/DEFORM
- **roahmlab/DEFT** — Branched-DLO differentiable DER follow-up | Updated: 2025 | https://github.com/roahmlab/DEFT
- **StructuresComp/dismech-rods** — C++ DDG/DER simulator with implicit integration, RA-L 2024; rod-shell coupling | ⭐47 | Updated: 2025 | https://github.com/StructuresComp/dismech-rods
- **StructuresComp/dismech-python** — Pure-Python re-implementation (Py-DiSMech) | Updated: 2025 | https://github.com/StructuresComp/dismech-python
- **QuantuMope/dismech-rl** — RL training stack on DisMech (SAC, 500 parallel envs, target-following + 3D obstacle reach) | Updated: 2024 | https://github.com/QuantuMope/dismech-rl
- **google-deepmind/mujoco** — Ships cable composite plugin (model/plugin/elasticity/cable.xml) — 41-segment capsule chain with bend/twist stiffness, ideal for rope/whip DLOs; Apache-2.0 | ⭐13.4k | Updated: 2026-04 (v3.8.0) | https://github.com/google-deepmind/mujoco
- **google-deepmind/mujoco_playground** — MJX GPU RL framework, RSS 2025 Outstanding Demo; no rope env yet | ⭐1.9k | Updated: 2026-03 | https://github.com/google-deepmind/mujoco_playground
- **GazzolaLab/PyElastica** — Cosserat-rod simulator (Python); gym-softrobot wrapper exists | ⭐335 | Updated: 2025-08 | https://github.com/GazzolaLab/PyElastica
- **Genesis-Embodied-AI/genesis-world** — Multi-physics GPU engine: PBD rope/cloth + MPM; 43M FPS claim | ⭐28.6k | Updated: 2025 | https://github.com/Genesis-Embodied-AI/genesis-world
- **dfki-ric/movement_primitives** — Best-maintained DMP / ProMP / bimanual-coupled-DMP toolkit; methodological match for whip-DMP optimization; BSD-3 | ⭐290 | Updated: 2025-05 | https://github.com/dfki-ric/movement_primitives
- **stulp/dmpbbo** — C++/Python DMP + black-box-optimization (CMA-ES) | ⭐~150 | https://github.com/stulp/dmpbbo
- **MosesAndLily/whip-project-targeting** — MuJoCo + NLopt classical optimization for 4-DoF arm + 50-DoF whip target-hitting; backs Nah 2023 iScience | ⭐7 | Updated: 2023 | https://github.com/MosesAndLily/whip-project-targeting
- **dondestamos/WhipTask_PerformanceWhipHand** — Krotov 2022 RSOS human whip-target experiment data + analysis | https://github.com/dondestamos/WhipTask_PerformanceWhipHand
- **elpis-lab/ActivePusher** — Residual-physics + NTK active-learning for nonprehensile manipulation | https://github.com/elpis-lab/ActivePusher
- **rail-berkeley/serl** (cable-routing branch) — Real-Franka SERL with Cable Routing task on hardware | ⭐1.6k+ | Updated: 2025 | https://github.com/rail-berkeley/serl

## Open Problems Being Attacked

- **Hardware-validated diffusion policy for whipping**. DIDP shows diffusion + test-time adaptation tops sim baselines on the 3D rope-whip benchmark, but has not been deployed on real hardware. NVIDIA SoftMimicGen and the Edinburgh distributional Real2Sim2Real are early attempts at the data side; no end-to-end real-rope diffusion policy has been published.
- **Closing the sim-to-real gap for fast (whip-like, > 5 m/s tip) DLO motions**. PRC and Free-End Cable transfer for slow casts; IRP transfers via iterative residuals; nothing transfers cleanly for whip-cracks. Active 2026 attacks: RAPiD's RMA-style particle embedding (UT-Austin/Stanford), Kamaras LFI distributional (Edinburgh), Gaussian-splatting real-to-sim eval (Columbia/DeepMind).
- **The 1-cm tip threshold on the 3D rope-whip benchmark**. DIDP hits 20.8 % at 1 cm — wide-open headroom; no other published method.
- **Combining the legged-throwing residual architecture with whipping**. ETH 2025 IROS 100 Hz nominal + 400 Hz residual is the obvious template; nobody has applied it to a robot holding a whip.
- **Single-arm vs bimanual whipping**. SoftMimicGen explicitly enumerates bimanual + humanoid + surgical embodiments for dynamic whipping; no published bimanual whip-to-target benchmark numbers yet.
- **VLM / language goal-conditioning for whipping**. Hierarchical DLO Routing (UMN 2025) uses VLMs for routing only; no work conditions a whip-tip target on language. Open niche.
- **Real-DLO validation tolerance for high-fidelity Cosserat sims**. DEFORM, DEFT, MuJoCo-DER (IROS 2025) validate static / quasi-static; no published rope simulator has sim-to-real validation specifically on whip-like (> 5 m/s tip-velocity) motions. Open gap directly relevant to whip-to-target.
- **TossingBot-style scalar residual on a closed-form rope prior**. Not yet attempted. The cleanest unexploited transfer from rigid throwing — replace TossingBot's ballistic `v̂` with an analytic Cosserat-derived end-to-tip transfer at the snap instant.

## Research Dynamics

The field velocity is *strongly accelerating* on three fronts: (1) bleeding-edge 2026 arXiv preprints (SoftMimicGen, RopeDreamer, RAPiD, Kamaras distributional R2S2R, FLASH, SIM1) are all from January–April and all touch DLO sim2real adaptation; (2) the throwing analog has converged on a clean two-rate residual recipe (TossingBot scalar residual → ETH continuous-trajectory residual + pullback-tube acceleration → ActivePusher active learning) that has not been applied to ropes; (3) differentiable Cosserat-rod stacks are maturing fast (DEFORM 2024 → DEFT 2025 → MAT-DiSMech 2026 IJRR → Py-DiSMech 2025) and now have hooks to PyTorch / JAX / MuJoCo. What is *stalling*: classical bullwhip analytical control beyond Goriely-McMillen — the Sternad/Hogan group is the only active classical thread, and it has converged on a 9-parameter min-jerk primitive plus DIRECT-L (Nah 2023, Edraki 2024); nobody is producing new physics. What is *fragmenting*: target encoding — 2025–2026 papers split between 3D-point goals (DIDP, IRP, MC-PILOT, ETH whole-body), full-shape image / polyline goals (Self-Curriculum-MBRL, Laezza), goal images (Takahashi 2023), and language sub-goals (Hierarchical DLO Routing). The likely next breakthrough: a hardware-validated **TossingBot-for-rope** — scalar-residual-on-Cosserat-prior or low-rate primitive + high-rate residual whipping, riding the simulator-stack maturity inflection.

## Cross-field Signals

- **Diffusion policies (2024 imitation-learning standard)** crossed into DLO manipulation in 2025 via DIDP and the Real-to-Sim Gaussian Splatting evaluator. The next transplant is likely diffusion-policy-as-residual-on-primitive (replacing Gaussian / NN heads in IRP / Free-End Cable).
- **RMA / particle-embedding sim2real adaptation (originally legged-locomotion, Kumar et al. 2021)** crossed into DLO mobile manipulation in March 2026 with RAPiD. Drop-in candidate for whip target-reaching.
- **Gaussian splatting (2023 vision standard)** crossed into robot policy evaluation in 2026 (Real2Sim Gaussian Splatting, ICRA 2026), explicitly listing rope routing as a benchmark task. Visual-fidelity sim2real for ropes is now feasible without rebuilding scene geometry.
- **Likelihood-free / simulation-based inference (originally cosmology / particle physics)** crossed into DLO sim2real adaptation via Edinburgh's distributional R2S2R (RA-L 2025 / arXiv 2502.18615 v4). Posterior over rope params replaces blind domain randomization.

## Specific Answer

**Q: How is goal-conditioned dynamic rope-tip target-reaching being learned today, and what simulators / methodological analogs from goal-conditioned throwing apply? Specifically, for each leading paper, what is the task setup, sim vs real, learning method, and action representation?**

The 2025–2026 working real-hardware recipe is **a low-dimensional parameterized swing primitive (5–10 parameters, joint-space min-jerk or task-space waypoints) refined by one of three update rules: (i) iterative residual on observed tip trajectories (IRP, ILC flying-knot), (ii) derivative-free / Bayesian black-box optimization over the primitive params (Nah 2023, Edraki 2024, Takahashi 2023), or (iii) one-shot supervised regression from sim trajectories (Free-End Cable 2024, Planar Robot Casting 2022)**. The 3D target is conditioned directly as a coordinate input. The only 2025–2026 published method that abandons the low-D primitive in favor of a full diffusion-policy reduced-order joint trajectory is DIDP (arXiv 2505.17434), which is sim-only and reports 3.6 cm mean tip error (84.3 % within 5 cm, 20.8 % within 1 cm) on its own benchmark. The dominant sim2real strategy on real hardware is still **IRP-style delta-dynamics + iterative residual**, which transfers without explicit domain randomization and across embodiments by modeling only relative changes induced by action perturbations.

| Paper | Task setup | Sim vs real | Learning method | Action representation |
|---|---|---|---|---|
| **IRP (Chi RSS 2022)** | Whip rope tip to 3D target (also cloth-to-pose); UR5 + Sawyer | Sim-trained, **zero-shot real UR5 + Sawyer** | Implicit policy via delta-dynamics network; adaptive action sampling at test time | Parameterized open-loop swing trajectory (~5–10 params, joint-space) + iterative residual on the param vector |
| **DIDP (Lan 2025, arXiv 2505.17434)** | 3D rope-tip target on 21-material-point reduced-order rope; T = 0.5 s | **Sim only** (Cosserat-rod, 55k trajectories) | Imitation-pretrained Diffusion Policy + physics-informed test-time adaptation enforcing kinematic + dynamics priors during denoising | Full reduced-order joint trajectory (20 DoF × T) — no primitive |
| **Robots of the Lost Arc (Zhang ICRA 2022)** | Whip cable arc to vault/knock/weave; UR5 | Real only | CNN over RGB-D → 3D apex point | 3D apex point → min-jerk QP arc (3-param plan) |
| **Planar Robot Casting (Lim ICRA 2022)** | Single-step planar rope cast to 2D target | Real2Sim2Real (sim params fitted to real, train sim, deploy) | GP / NN regressor on (rope-state, target) → trajectory | Single-step planar EE trajectory params (open-loop, low-D) |
| **Free-End Cable (Wang 2024, arXiv 2405.09581)** | Single-step planar free-tip cable cast | PyBullet differentially tuned to real, deploy real UR5 | Self-supervised regression | Single-step trajectory (low-D, joint-space) |
| **SDU whip-targeting (SIMPAR 2025)** | 7-DoF arm whips rope to 3D target in <1.5 s | MuJoCo + OIAC sim2real | SAC / PPO RL + Online Impedance Adaptation Controller | Joint-space motion-plan parameters |
| **Implicit Physics-aware Policy (Wang ICRA 2025, arXiv 2502.05696)** | Goal-conditioned **one-shot** rope-as-tool transport of rigid object to 3D target | Sim-train, real-deploy | Implicit sysID encoder + goal-conditioned action predictor; no test-time loop | Single end-effector trajectory |
| **ILC flying-knot (CMU 2026, arXiv 2602.21302)** | Non-planar arcing strike to flip rope through a self-formed loop; xArm 7 | **Real hardware only** | Task-Level Iterative Learning Control from a single human demo + simplified rope model | Parameterized handle trajectory + ILC-update vector |
| **Nah iScience 2023** | Whip rope to 3D target in MuJoCo, 4-DoF arm + 50-DoF segmented whip | Sim only | DIRECT-L black-box opt over a 9-param primitive | 9-param min-jerk submovement (joint-space) |
| **Edraki BioRob 2024** | Preparing-and-striking whip handle to 3D target; 25-link 1 m whip | Sim+small physical | MATLAB pattern-search black-box opt | 5–9-param min-jerk handle trajectory (task-space) |
| **TossingBot (Zeng RSS 2019 / T-RO 2020)** | Pick + throw arbitrary objects to 3D bin targets; UR5 + gripper | Real for everything; PyBullet only for ballistic prior | FCN over heightmap; residual-physics; self-supervised on landing position | **Scalar residual on tangential release velocity**; angle fixed at 45°; goal encoded by broadcasting `v̂` as 128-channel feature image |
| **MC-PILOT (Turcato 2025, arXiv 2502.05595)** | Throw to 3D target; Franka Panda | Real | Probabilistic MBRL (PILCO-style) with explicit release-delay model | Release position + 3-D release velocity (continuous) |
| **ETH Whole-Body Throwing (Ma IROS 2025, arXiv 2506.16986)** | Legged manipulator throws to 3–6 m 3D target; ANYmal-D + DynaArm | Sim-trained, real-deployed | 100 Hz nominal MPC + 400 Hz RL residual + pullback-tube acceleration optimizer | EE-velocity reference + joint-level residuals (continuous) — 0.276 m mean error @ 6 m |
| **Throw-Flip (Liu/Billard 2025, arXiv 2510.10357)** | Throw object to target 6-DoF pose; revolute arm | Real | Impulse-momentum decoupling + regression residual on free-flight | Release linear vel + release angular vel (decoupled) |
| **Ploeger juggling (CoRL 2020, arXiv 2010.13483)** | Two-ball juggling on Barrett WAM | Real only | Episodic policy search with binary catch reward | Compact parameterized motor-primitive (DMP-like via-points) |
| **Kober dart-throwing (Auro 2012)** | Dart-throw + ball-paddling on real WAM / BioRob / CBi / Kuka | Real | Cost-regularized kernel regression / reward-weighted regression on DMP meta-params | DMP global parameters (release angle + release velocity) |

**Throwing → rope methodological transfers worth attempting.** (1) **TossingBot-for-rope**: replace TossingBot's ballistic `v̂` with an analytic min-jerk / Cosserat-derived end-to-tip transfer at the snap instant; predict a scalar residual on amplitude/angle. This is the cleanest unexploited transfer and the natural successor to IRP — it would replace IRP's iterative residual with a one-shot residual conditioned on the goal-encoded prior. (2) **MC-PILOT-style release-delay model** maps directly to the whip-snap delay between handle motion peak and tip arrival, currently absorbed implicitly into IRP's delta-dynamics. (3) **ETH's 100 Hz nominal + 400 Hz residual** maps to a base-trajectory primitive + high-rate joint residual for whip targeting on a mobile or floating base (e.g., quadruped + arm). (4) **Throw-Flip's impulse-momentum decoupling** (linear release + angular release independently) is a candidate model for separating handle linear motion (sets tip arrival point) from wrist-flick angular impulse (sets traveling-wave energy down the rope).

**Simulator-stack recommendation.** For a new sim2real project on rope-tip target reaching the recommended stack is **DEFORM (CoRL 2024, differentiable PyTorch DER)** for gradient-based training, **DIDP's reduced-order 20-DoF Cosserat sim** if you want to compete on the published 3D rope-whip benchmark, and **DaXBench WhipRope (JAX, ICLR 2023)** for parallel RL with the existing WhipRope task and APG baseline at 0.83. For sim2real, layer **Real2Sim2Real Differential-Evolution fitting** (the PRC recipe) — it remains the only published recipe with reliable transfer for fast DLO motions, since domain randomization alone has not been shown to work for whipping. **MuJoCo-DER generalized coordinates (IROS 2025)** is the right choice if rope is one of several objects in a larger scene, since MJX gives GPU-batched rollouts. **First reproduction target should be IRP on UR5** (code at https://github.com/columbia-ai-robotics/irp), then layer DIDP's diffusion + test-time adaptation on top *as a residual over IRP's primitive*, not as a replacement — empirically obvious next paper.

## Must-Read Papers (Last 12 Months)

- **[Dynamic Manipulation of Deformable Objects in 3D (DIDP)](https://arxiv.org/abs/2505.17434)** — first dedicated 3D rope-whip benchmark with concrete numbers (3.6 cm mean, 20.8 % at 1 cm); diffusion-policy + physics-informed test-time adaptation on a 20-DoF reduced-order Cosserat sim.
- **[Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration](https://arxiv.org/abs/2506.16986)** — IROS 2025 ETH; 100 Hz nominal + 400 Hz residual whole-body throwing template directly applicable to whip targeting.
- **[Implicit Physics-aware Policy for Dynamic Manipulation of Rigid Objects via Soft Body Tools](https://arxiv.org/abs/2502.05696)** — ICRA 2025; one-shot rope-as-tool 3D target transport, explicitly contrasts with IRP's iterative loop.
- **[Learning Deformable Object Manipulation Using Task-Level ILC (flying-knot)](https://arxiv.org/abs/2602.21302)** — Feb 2026; 100 % success in ≤ 10 trials on real xArm 7 across 7 ropes, ILC from a single human demo + simplified model.
- **[SoftMimicGen](https://arxiv.org/abs/2603.25725)** — Mar 2026 NVIDIA / Mandlekar; first scaled data-generation system to enumerate dynamic whipping as a benchmark behavior across single-arm, bimanual, humanoid, surgical embodiments.
- **[RopeDreamer](https://arxiv.org/abs/2604.28161)** — Apr 2026 TU Darmstadt / Honda; Dreamer-style RSSM with quaternionic kinematic latent that intrinsically preserves rope inextensibility; 40.5 % rollout-error reduction over 50 steps.
- **[RAPiD: Rapid Adaptation of Particle Dynamics](https://arxiv.org/abs/2603.18246)** — Mar 2026 UT-Austin / Stanford; RMA-style particle-embedding sim-to-real for deformable mobile manipulation; > 80 % real-world success on 2 tasks.
- **[Self-Curriculum Model-based RL for Shape Control of DLOs](https://arxiv.org/abs/2602.21816)** — Feb 2026 Tsinghua; model-based RL + HER curriculum + Jacobian visual servoing; 30/30 zero-shot real-world success across DLO size and material.

## Cached sources

- [[l1a-1-iterative-residual-policy-rope-whip]]
- [[l1a-2-didp-dynamic-manipulation-3d]]
- [[l1a-3-free-end-cable-dynamic-planar]]
- [[l1a-4-planar-robot-casting-real2sim2real]]
- [[l1a-5-drl-whip-targeting-simpar2025]]
- [[l1b-1-dexdlo-goal-conditioned-dexterous-dlo]]
- [[l1b-2-laezza-offline-gcrl-dlo-shape]]
- [[l1b-3-3d-dynamic-deformable-benchmark-didp]]
- [[l1b-4-self-curriculum-mbrl-dlo-shape]]
- [[l1b-5-hierarchical-dlo-routing-rl-vlm]]
- [[l1b-6-huo-rearranging-dlo-implicit-goals]]
- [[l1c-1-tossingbot-residual-physics]]
- [[l1c-2-mc-pilot-data-efficient-throwing]]
- [[l1c-3-whole-body-throwing-residual-pullback]]
- [[l1c-4-throw-flip-billard]]
- [[l1c-5-juggling-binary-rewards-ploeger]]
- [[l1d-1-krotov-motor-control-beyond-reach-whip]]
- [[l1d-2-nah-learning-whip-primitive-actions]]
- [[l1d-3-edraki-human-inspired-whip-preparatory]]
- [[l1d-4-goriely-mcmillen-shape-cracking-whip]]
- [[l1d-5-krehl-puzzle-whip-cracking]]
- [[l1e-1-deform-differentiable-der]]
- [[l1e-2-daxbench-whiprope]]
- [[l1e-3-genesis-embodied-ai]]
- [[l1e-4-mujoco-der-generalized]]
- [[l1e-5-dismech-soft-robot-sim]]
- [[l2a-1-softmimicgen-deformable-data-gen]]
- [[l2a-2-real2sim2real-distributional-dlo]]
- [[l2a-3-ropedreamer-rssm-quaternionic]]
- [[l2a-4-rapid-particle-dynamics-mobile]]
- [[l2a-5-hybrid-deformable-rigid-cosserat]]
- [[l2b-1-didp-3d-rope-whip-benchmark]]
- [[l2b-2-daxbench-whiprope-baselines]]
- [[l2b-3-planar-robot-casting-tables]]
- [[l2b-4-tossingbot-baseline-tables]]
- [[l2b-5-whole-body-throwing-ablation]]
- [[l2c-1-implicit-physics-aware-soft-tools]]
- [[l2c-2-whole-body-throwing-pullback-tube]]
- [[l2c-3-activepusher-residual-active-learning]]
- [[l2c-4-real2sim-gaussian-splatting-rope-routing]]
- [[l2c-5-didp-3d-deformable-benchmark]]
- [[l2d-1-dismech-rods]]
- [[l2d-2-mujoco-cable-plugin]]
- [[l2d-3-pyelastica]]
- [[l2d-4-deft-roahmlab]]
- [[l2d-5-movement-primitives-dfki]]
- [[l2e-1-irp-task-and-action-spec]]
- [[l2e-2-didp-numbers-and-action]]
- [[l2e-3-tossingbot-action-and-goal-encoding]]
- [[l2e-4-rope-tip-recipe-synthesis]]

**Total**: `raw/agentic-search/2026-05-06-rope-whip-target-reaching-deep/` (49 files)
