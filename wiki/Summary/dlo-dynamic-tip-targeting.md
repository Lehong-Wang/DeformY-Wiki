---
title: "Goal-Conditioned Dynamic Rope/DLO Tip Targeting"
scope: "Robot-arm dynamic manipulation of ropes and other deformable linear objects toward arbitrary tip-position targets, with learned policies; the simulators and methodological analogs that make it work; the gap that defines the open research direction."
key_topics:
  - rope-tip-targeting
  - iterative-residual-policy
  - residual-physics
  - dynamics-informed-diffusion-policy
  - differentiable-discrete-elastic-rods
  - real2sim2real-pipeline
  - cosserat-rod-theory
  - sim-to-real-transfer
  - apex-point-trajectory-parameterization
  - task-level-iterative-learning-control
  - high-frequency-residual-policy
  - implicit-system-identification
  - task-agnostic-system-identification
paper_count: 18
date_updated: 2026-05-06
sources:
  - raw/notes/dlo-tip-targeting-survey-claude.md
  - raw/notes/dlo-tip-targeting-survey-agentic.md
  - raw/notes/dlo-tip-targeting-survey-gpt.md
---

## Overview

Goal-conditioned dynamic manipulation of ropes — whipping a free-tip rope so it reaches a designated 3D target — is a small but rapidly consolidating literature. As of May 2026 it is anchored by **[[iterative-residual-policy-goal-conditioned-dynamic]]** (Chi et al., RSS 2022 / IJRR 2024) and is being extended along three orthogonal axes: a 2025–2026 surge of one-shot and zero-shot real-hardware methods (**[[implicit-physics-aware-policy-dynamic-manipulation]]**, **[[learning-deformable-object-manipulation-using-task]]**, **[[wiggle-go-system-identification-zero-shot]]**), the first dedicated 3D rope-whip benchmark with leaderboard numbers (**[[dynamic-manipulation-deformable-objects-3d-simulation]]** — DIDP), and a maturing differentiable-Cosserat-rod simulation stack (**[[deform-differentiable-discrete-elastic-rods-real]]**, **[[accurate-simulation-parameter-identification-dlos-using]]**, **[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]**). Methodological transfer from goal-conditioned rigid throwing — anchored by **[[tossingbot-learning-throw-arbitrary-objects-residual]]** and generalized in 2025 by **[[learning-accurate-whole-body-throwing-high]]** — supplies the dominant architectural template (low-rate analytic prior + high-rate learned residual) that has not yet been transferred end-to-end to whipping.

**The gap thesis** ([[no-paper-has-real-3d-arbitrary-target-learned-policy-whipping]]): no published paper as of May 2026 simultaneously delivers all four of {real-robot hardware, arbitrary 3D target, learned runtime policy, free-space dynamic whipping}. Each property is individually realized — IRP is real-hw + learned-policy but 2D-plane; DIDP is 3D + learned but sim-only; Wiggle-and-Go is real-hw + 3D but uses CMA-ES trajopt rather than a learned policy; IPA is real-hw + 3D + learned but conditions on rigid-payload landing, not free-tip. The intersection is the niche.

## Core areas

### 1. Direct rope/DLO + dynamic + learned + 3D-target (anchor of the literature)

- **[[iterative-residual-policy-goal-conditioned-dynamic]]** (Chi RSS 2022 / IJRR 2024) — UR5 whips a rope to a 3D target via a learned **[[delta-dynamics-network]]** + sampling-based action refinement on a low-D parameterized swing primitive. Verified target space is 2D Y–Z plane on real hardware. Sim-to-real-zero-shot across multiple ropes and embodiments. RSS 2022 Best Paper.
- **[[dynamic-manipulation-deformable-objects-3d-simulation]]** (Lan et al. 2025, DIDP) — first 3D rope-whip benchmark with leaderboard. DIDP is **[[dynamics-informed-diffusion-policy]]** + **[[physics-informed-test-time-adaptation]]** on a 20-DoF reduced-order Cosserat sim. Sim-only, 84.3% within 5 cm and 20.8% within 1 cm.
- **[[implicit-physics-aware-policy-dynamic-manipulation]]** (Wang & Qureshi 2025, IPA) — one-shot rope-as-tool 3D-target transport of rigid payloads. Replaces IRP's iterative residual loop with up-front **[[implicit-system-identification]]**. Real UR5e, 62.5% real-world success.
- **[[learning-deformable-object-manipulation-using-task]]** (Suresh & Atkeson, CMU 2026) — Task-Level ILC on real xArm 7 for the flying-knot task. 100% success in ≤10 trials across 7 rope/cable types. Conceptually descended from IRP via the **[[task-level-iterative-learning-control]]** framing with **[[critical-point-objective]]**.
- **[[wiggle-go-system-identification-zero-shot]]** (CMU 2026) — closest published match to the gap-thesis target task. **[[task-agnostic-system-identification]]** via a single safe wiggle observation, then CMA-ES trajectory optimization in Drake at task time. Zero-shot real-hardware 3D point striking on xArm 7.
- **[[robots-lost-arc-self-supervised-learning]]** (Zhang et al., ICRA 2022) — 3D **[[apex-point-trajectory-parameterization]]** for fixed-endpoint cable manipulation. Anchor of one of the three canonical action representations.

### 2. Adjacent: planar / fixed-endpoint dynamic cable target reaching

- **[[planar-robot-casting-real2sim2real-self-supervised]]** (Lim et al., ICRA 2022) — **[[real2sim2real-pipeline]]** with **[[differential-evolution-sim-tuning]]** and a parameterized two-arc primitive on UR5. Per-cable median tip-error 8 / 12 / 14 % of cable length on planar casting beyond workspace.
- **[[self-supervised-learning-dynamic-planar-manipulation]]** (Wang et al. 2024) — direct extension to free-end cables. Introduces the **[[two-arc-planar-motion-primitive]]**. 22 / 24 / 34 % tip error on 3 real cables.
- **[[deformx-versatile-co-simulation-framework-deformable]]** (existing wiki paper) — Cosserat-rod + Isaac Sim **[[cosserat-isaac-cosimulation]]** framework that targets the same DLO sim-to-real surface from the data-side.

### 3. Throwing as methodological analog

- **[[tossingbot-learning-throw-arbitrary-objects-residual]]** (Zeng et al., RSS 2019 / T-RO 2020) — canonical **[[residual-physics]]** paper. Scalar residual on tangential release velocity over an analytic ballistic prior; 84.7% real-world throw accuracy on seen objects.
- **[[learning-accurate-whole-body-throwing-high]]** (Ma et al., IROS 2025) — generalizes TossingBot's scalar residual to a continuous 100 Hz nominal MPC + 400 Hz **[[high-frequency-residual-policy]]** + **[[pullback-tube-acceleration]]** stack. ANYmal-D + DynaArm; 0.276 m mean landing error at 6 m. The cleanest unexploited transfer template for whipping.

### 4. Simulators (the substrate that makes any of the above trainable)

- **[[deform-differentiable-discrete-elastic-rods-real]]** (Chen et al., CoRL 2024) — **[[differentiable-discrete-elastic-rods]]** with a **[[neural-residual-on-physics-model]]**, validated on real industrial DLOs.
- **[[accurate-simulation-parameter-identification-dlos-using]]** (Chen, Bretl, Pham, IROS 2025) — Bergou DER bending+twisting energies inside MuJoCo's generalized-coordinate solver via **[[der-mujoco-generalized-coordinate-coupling]]**. MJX-compatible, GPU-batched.
- **[[daxbench-benchmarking-deformable-object-manipulation-differentiable]]** (Chen et al., ICLR 2023 Oral) — JAX **[[differentiable-deformable-benchmark]]** with the WhipRope task built in. APG 0.83 vs SHAC 0.66 vs PPO 0.25 baselines.

### 5. Bleeding-edge 2026 sim-to-real adaptation

- **[[softmimicgen-data-generation-system-scalable-robot]]** (NVIDIA + UT-Austin 2026) — **[[automated-demo-generation-deformable]]**; first scaled data-gen system to enumerate dynamic whipping as a benchmark behavior across single-arm, bimanual, humanoid, surgical embodiments.
- **[[ropedreamer-kinematic-recurrent-state-space-model]]** (TU Darmstadt + Honda 2026) — Dreamer-style **[[quaternionic-rssm-dlo]]** that preserves rope inextensibility by construction. 40.5% rollout-error reduction over baseline RSSM.
- **[[rapid-adaptation-particle-dynamics-generalized-deformable]]** (UT-Austin + Stanford 2026) — RMA-style **[[rma-particle-dynamics-adaptation]]** with privileged particle-position teacher → depth-policy student. >80% real-world success on 22-DOF mobile manipulation.
- **[[self-curriculum-model-based-reinforcement-learning]]** (Tsinghua 2026) — Two-stage **[[self-curriculum-goal-generation]]** MBRL + Jacobian visual servoing. 30/30 zero-shot real-world success on DLO shape control.

## Evolution

| Year | Anchor result | What changed |
|---|---|---|
| 2008 | Bergou DER (foundation) | Discrete elastic rods become the canonical rope-physics formulation |
| 2019 | TossingBot | Residual physics on top of an analytic prior beats both pure-physics and pure-learning for goal-conditioned throwing |
| 2022 | IRP / Real2Sim2Real / Lost-Arc | Three converging recipes for goal-conditioned dynamic cable: iterative residual on a primitive (IRP), DE-tuned simulator + supervised regression (R2S2R), apex-point arc (Lost-Arc) |
| 2023 | DaXBench / DER-MuJoCo (preprint) | Differentiable rope-physics benchmarks become available |
| 2024 | DEFORM / Free-End-Cable | Differentiable DER + neural residual; free-tip target reaching extends Real2Sim2Real |
| 2025 | DIDP / IPA / ETH-WB-Throw | First 3D rope-whip benchmark; one-shot rope-as-tool; whole-body residual stack |
| 2026 | Wiggle&Go / Flying-Knot-ILC / 4 preprints | Real-hardware 3D rope-strike via sysID-then-trajopt; ILC on real flying-knot; data-gen / world-models / adaptation surge |

The trend line is clear: sim-side maturity (DEFORM 2024 → DEFT 2025 → MAT-DiSMech 2026 → DaXBench WhipRope), data-side scaling (SoftMimicGen 2026), action-rep convergence on either *low-D primitive + residual* (the throwing line) or *full reduced-order joint trajectory + diffusion* (DIDP), and an explicit gap on the real-hardware-3D-learned-policy intersection.

## Current frontiers

- **Hardware-validated diffusion policy for whipping** — DIDP is sim-only; SoftMimicGen and RAPiD are early data-side attempts; no end-to-end real-rope diffusion policy yet.
- **Closing the sim-to-real gap for fast (whip-like, > 5 m/s tip) DLO motions** — domain randomization alone has not been shown to work; Real2Sim2Real and IRP iterative residuals work for slower regimes; RAPiD's RMA-style particle embedding and Edinburgh's distributional R2S2R are 2026 attempts.
- **Sub-1-cm tip threshold on the 3D rope-whip benchmark** — DIDP hits 20.8% at 1 cm; wide-open headroom.
- **TossingBot-for-rope** — replace TossingBot's ballistic `v̂` with an analytic Cosserat-derived end-to-tip transfer at the snap instant; predict a scalar residual on amplitude/angle. Cleanest unexploited transfer.
- **ETH 100 Hz nominal + 400 Hz residual on a robot holding a whip** — direct architectural transfer; not yet attempted.
- **Real-DLO validation tolerance for high-fidelity Cosserat sims at whip-class speeds** — DEFORM, DEFT, DER-MuJoCo all validate static / quasi-static; no published rope simulator has sim-to-real validation specifically on > 5 m/s tip-velocity.
- **Single-arm vs bimanual whipping** — SoftMimicGen explicitly enumerates bimanual + humanoid + surgical embodiments; no published bimanual whip-to-target benchmark yet.
- **VLM / language goal conditioning for whipping** — Hierarchical DLO Routing (UMN 2025) uses VLMs for routing only; no work yet conditions a whip-tip target on language.

## Recommendations

Synthesized from the three surveys' recommendations sections:

1. **Reproduce IRP on UR5 first** — it is the published real-hardware floor and gives the right action parameterization (low-D swing parameters) to compare against. Code is on the Columbia AI Robotics page.
2. **For simulation, pick DER, not PBD/XPBD** — DER ([[deform-differentiable-discrete-elastic-rods-real]] or [[accurate-simulation-parameter-identification-dlos-using]]) captures bending+twisting at high speed. PyElastica is the alternative for Python-native Cosserat. Avoid linear-stiffness rope models (basic mass-spring, MuJoCo native composite) for whipping.
3. **Use a structured action representation** — apex points, motion-manifold latents, or DMP/MMP trajectory parameters are more sample-efficient and transferable than full RL over joints. DIDP is the only published counter-example; treat it as a research direction, not a default.
4. **Build in online adaptation** — pure open-loop sim-to-real is fragile across rope diversity. Plan for at minimum a system-ID probe (Wiggle&Go-style) + 1–10 real refinement iterations (IRP-style).
5. **Borrow residual physics from TossingBot** — overlay a learned residual on a Cosserat-derived feed-forward trajectory. Combines analytic generalization with data-driven correction; analogous to [[high-frequency-residual-policy]] + [[pullback-tube-acceleration]] from the ETH whole-body throwing line.

### Decision thresholds

- **<5 cm tip error at high speed required** → DER/Cosserat sim mandatory; abandon linear-stiffness ropes.
- **≥1 kHz visual feedback hardware available** → benchmark against Yamakawa-Ishikawa high-speed-vision approach as a non-learning baseline.
- **One-shot only (no per-target iteration)** → IPA / DA-MMP / motion-manifold methods over IRP-style iterative residual.
- **Targets in 3D with obstacles** → Lost-Arc 3D apex-point representation.
- **Heterogeneous payload on rope tip** → IPA is currently the only paper tackling this regime.

## Key references

### Top-17 (full wiki ingest)

- [[iterative-residual-policy-goal-conditioned-dynamic]]
- [[tossingbot-learning-throw-arbitrary-objects-residual]]
- [[deform-differentiable-discrete-elastic-rods-real]]
- [[accurate-simulation-parameter-identification-dlos-using]]
- [[planar-robot-casting-real2sim2real-self-supervised]]
- [[dynamic-manipulation-deformable-objects-3d-simulation]]
- [[learning-accurate-whole-body-throwing-high]]
- [[implicit-physics-aware-policy-dynamic-manipulation]]
- [[learning-deformable-object-manipulation-using-task]]
- [[daxbench-benchmarking-deformable-object-manipulation-differentiable]]
- [[softmimicgen-data-generation-system-scalable-robot]]
- [[ropedreamer-kinematic-recurrent-state-space-model]]
- [[rapid-adaptation-particle-dynamics-generalized-deformable]]
- [[self-curriculum-model-based-reinforcement-learning]]
- [[wiggle-go-system-identification-zero-shot]]
- [[robots-lost-arc-self-supervised-learning]]
- [[self-supervised-learning-dynamic-planar-manipulation]]
- [[deformx-versatile-co-simulation-framework-deformable]] (existing — Cosserat-Isaac cosimulation)

### Bibliography (notes-only, 77 entries in `raw/notes/papers/`)

Selected highlights of papers covered in the bibliography but not given full wiki pages — see `raw/notes/papers/INDEX.md` for the full list:

- **Whip physics**: `bullwhip-physics-goriely-mcmillen.md`, `krehl-puzzle-whip-cracking.md`
- **Human-motor-control whip cluster (Sternad/Hogan/Northeastern–MIT)**: `krotov-motor-control-beyond-reach-whip.md`, `nah-learning-whip-primitive-actions.md`, `nah-biorob-2020-dynamic-primitives-whip.md`, `edraki-human-inspired-whip-preparatory.md`, `xiong-online-impedance-adaptation-whip.md`, `wang-xiong-biorob-2024-whip-control.md`
- **Classical Yamakawa line**: `yamakawa-tokyo-high-speed-rope-knotting-series.md`
- **Cosserat / DER simulators**: `discrete-elastic-rods-bergou-2008.md`, `corde-cosserat-rod-elements.md`, `elastica-pyelastica-naughton.md`, `dismech-rods-jawed-2024.md`, `deft-roahmlab-branched-der.md`, `tong-jawed-sim2real-dlo-ijrr.md`
- **DLO simulators (non-DER)**: `mujoco-cable-plugin.md`, `genesis-embodied-ai.md`, `isaac-lab-physx5.md`, `newton-nvidia-deepmind-unified-solver.md`, `difftaichi-ICLR-2020.md`, `nimble-physics-werling-icra-2021.md`, `pbd-xpbd-foundations.md`, `diff-soft-body-simulator-cluster.md`
- **Adjacent rope-learning (quasi-static / shape-control)**: `dexdlo-edinburgh-goal-conditioned-dexterous-dlo.md`, `laezza-offline-gcrl-dlo-shape.md`, `hierarchical-dlo-routing-rl-vlm.md`, `huo-rearranging-dlo-implicit-goals.md`, `yu-mingrui-whole-body-dlo-3d-constrained.md`, `genorm-one-shot-rope-parameter-aware.md`, `zimmermann-poranne-coros-implicit-integration.md`
- **Throwing & striking analogs**: `mc-pilot-data-efficient-throwing.md`, `da-mmp-motion-manifold-throwing.md`, `dmmp-differentiable-motion-manifold-primitives.md`, `whole-body-dynamic-throwing-legged.md`, `eth-anymal-badminton-2025.md`, `eth-excavator-dynamic-throwing.md`, `throw-flip-billard.md`, `juggling-binary-rewards-ploeger.md`, `kober-dart-throwing-meta-parameters-2012.md`, `dartbot-tactile-deformable-throwing.md`, `archery-icub-kormushev-calinon.md`
- **Cable / cloth foundational**: `flingbot-ha-song.md`, `dextairity-xu-rss2022.md`, `nair-self-supervised-rope-imitation-2017.md`, `wu-yan-deformable-without-demos.md`, `berkeley-autolab-untangling-cluster.md`, `sundaresan-grannen-cable-descriptors-cluster.md`, `she-tactile-cable-following-scirobot.md`, `kudoh-in-air-knotting-dual-arm-iros2021.md`, `takahashi-goal-image-cable-bayesian.md`
- **Real2Sim / sim2real adaptation**: `real2sim2real-distributional-dlo.md`, `real2sim-gaussian-splatting-rope-routing.md`, `diffcloud-sundaresan-real2sim.md`, `soma-real-to-sim-neural-simulator.md`, `active-pusher-residual-active-learning.md`, `hybrid-deformable-rigid-cosserat.md`

Two papers were considered for ingest but had no open-access source so are notes-only:
- `archery-icub-kormushev-calinon.md` — IEEE ICAR ~2010
- `zimmermann-poranne-coros-implicit-integration.md` — RA-L 2021

## Caveats

- **Several anchor 2025–2026 papers are arXiv preprints** (DIDP, IPA, Flying-Knot-ILC, Wiggle&Go, SoftMimicGen, RopeDreamer, RAPiD, Self-Curriculum-MBRL). Performance numbers are author-reported; peer-reviewed venue placement and independent reproduction are still in progress for several.
- **Bullwhip physics (Goriely/McMillen) and human motor control (Sternad/Hogan)** provide constraints/inspiration rather than methods. None of the surveyed robot-learning papers reproduces a true supersonic loop in simulation; all use sub-supersonic regimes.
- **The Yamakawa/Ishikawa line** is a classical non-learning baseline; its real-world success relies on 1 kHz vision and a specialized 4-DOF Barrett high-speed arm — adapting to standard 7-DOF arms (Franka, UR5) requires reformulation.
- **FlingBot and DextAIRity are cloth, not rope** — included as methodological analogs (pick-stretch-fling primitive) but they do not condition on a 3D point target.
- **Throwing analogs target moving objects or use whole-body motion** — transferring their RL-with-PPO-in-Isaac-Gym recipes to a deformable rope is non-trivial and has not yet been published end-to-end.

## Related

### Foundations leaned on heavily

- [[discrete-elastic-rods]]
- [[cosserat-rod-theory]]
- [[deformable-linear-object]]
- [[sim-to-real-transfer]]
- [[mass-spring-system]]
- [[position-based-dynamics]]
- [[model-based-reinforcement-learning]]
- [[diffusion-policy]]
- [[behavioral-cloning]]
- [[imitation-learning]]
- [[domain-randomization]]
- [[shape-servoing]]

### Concepts introduced or central to this synthesis

- [[iterative-residual-policy]]
- [[delta-dynamics-network]]
- [[residual-physics]]
- [[real2sim2real-pipeline]]
- [[differential-evolution-sim-tuning]]
- [[differentiable-discrete-elastic-rods]]
- [[der-mujoco-generalized-coordinate-coupling]]
- [[dynamics-informed-diffusion-policy]]
- [[physics-informed-test-time-adaptation]]
- [[high-frequency-residual-policy]]
- [[pullback-tube-acceleration]]
- [[implicit-system-identification]]
- [[task-level-iterative-learning-control]]
- [[task-agnostic-system-identification]]
- [[apex-point-trajectory-parameterization]]
- [[two-arc-planar-motion-primitive]]
- [[differentiable-deformable-benchmark]]
- [[quaternionic-rssm-dlo]]
- [[rma-particle-dynamics-adaptation]]
- [[self-curriculum-goal-generation]]

### Claims central to this synthesis

- [[no-paper-has-real-3d-arbitrary-target-learned-policy-whipping]] — the gap thesis
- [[irp-zero-shot-sim2real-rope-whipping]]
- [[didp-3d-rope-tip-targeting-success-rates]]
- [[hf-residual-tube-stack-enables-accurate-whole-body-throwing]]
- [[residual-physics-improves-throw-accuracy]]
- [[real2sim2real-prc-tip-error-8-14-percent]]
- [[free-end-cable-target-reaching-harder]]

### People (Berkeley AUTOLAB / CMU / ETH RSL / UMich ROAHM / etc.)

- [[cheng-chi]] [[shuran-song]] (IRP, Columbia/TRI)
- [[andy-zeng]] (TossingBot, Princeton/Google)
- [[ken-goldberg]] [[jeffrey-ichnowski]] [[harry-zhang]] [[vincent-lim]] [[jonathan-wang]] (Berkeley AUTOLAB / CMU)
- [[krishna-suresh]] [[chris-atkeson]] [[arthur-jakobsson]] (CMU Robotics Institute)
- [[yizhou-chen]] (UMich ROAHM, DEFORM/DEFT)
- [[yuntao-ma]] (ETH RSL, whole-body throwing + badminton)
- [[guanzhou-lan]] (NPU + Khalifa, DIDP)
- [[zixing-wang]] (Purdue, IPA)
- [[siwei-chen]] (NUS AdaComp, DaXBench)
- [[tim-missal]] [[jan-peters]] (TU Darmstadt, RopeDreamer)
- [[masoud-moghani]] (NVIDIA / U-Toronto, SoftMimicGen)
- [[qi-jing-chen]] (NTU + UIUC, DER-MuJoCo)
