---
title: "Accurate Simulation and Parameter Identification of Deformable Linear Objects using Discrete Elastic Rods in Generalized Coordinates"
slug: "accurate-simulation-parameter-identification-dlos-using"
arxiv: "2310.00911"
venue: "IROS"
year: 2025
tags: [DLO, deformable-linear-object, discrete-elastic-rods, mujoco, simulation, parameter-identification, bending-stiffness, twisting-stiffness]
importance: 4
date_added: 2026-05-06
source_type: tex
s2_id: "8a8bff1b084277bb3f681dd77c8156bc8596458a"
keywords: [DLO, Discrete Elastic Rods, MuJoCo, generalized coordinates, force-lever analysis, parameter identification, bending stiffness, twist stiffness, helical buckling, Michell's instability]
domain: "Robotics"
code_url: "https://github.com/qj25/adapteddlo_muj"
cited_by: []
---

## Problem

Modern robot-learning pipelines need DLO simulators that are simultaneously (i) physically faithful enough to match real elastic-rod statics under bending and twisting, (ii) fast enough for MPC and policy training, and (iii) embedded in a general-purpose physics simulator that already provides robots, contact, and a Python API. The two ends of the spectrum — accurate stand-alone DER/Cosserat solvers (e.g. PyElastica) and the native MuJoCo cable plugin — each fail one of those requirements. PyElastica gets the physics right but lives outside MuJoCo, so any robot-DLO interaction requires bespoke contact and integration glue. MuJoCo's native cable applies a *linear* torque-vs-curvature/twist law at each ball joint, which violates Kirchhoff rod theory at large deformations and exhibits unnatural twist-wave oscillations because it does not treat the material-frame twist quasi-statically. The paper argues this gap blocks both reliable parameter identification of real DLOs and reusable simulation for downstream robot learning.

## Key idea

Re-implement Bergou et al.'s **Discrete Elastic Rods** bending and twisting energies *inside* MuJoCo by mapping the DER Cartesian centerline forces into the existing **generalized (joint) coordinates** of a chain of MuJoCo capsules connected by ball joints. The mapping uses a **moment-based force-lever analysis**: convert the Cartesian stiffness force at every node into ball-joint torques via $\vec{\tau}_{i,j} = \vec{F}_{j}\times\vec{X}_{i,j}$, summed across all nodes. This avoids the $n+1$ Jacobian queries that would be needed if DER's Cartesian forces were applied directly to MuJoCo bodies. Combined with DER's **quasi-static treatment of the material-frame twist** (closed-form Gauss-Seidel-style update), the result is a MuJoCo-native rod model with DER-grade accuracy and only $\sim$1–3% computational overhead over the kinematic chain alone. A simple, hardware-light **parameter identification pipeline** (one bending experiment + one critical-twist experiment, depth camera, 3D-printed twist apparatus) then recovers $\alpha = EI$ and $\beta = GJ_T$ for real DLOs.

## Method

- **DER recap and stiffness moduli.** Vertices $\{x_i\}$, per-section twist $\theta^j$ relative to a Bishop frame; bending energy gradient $\partial E/\partial x_i \propto \alpha$, twist energy gradient $\partial E/\partial \theta^j \propto \beta$. The paper makes the (often-omitted) link explicit: $\alpha = EI$, $\beta = GJ_T$, where $E$ is Young's modulus, $G$ shear modulus, $I$ second moment of area, and $J_T$ torsion constant of the (constant) cross-section.
- **Generalized-coordinate adaptation (`adapted`).** DLO modeled as a kinematic chain of MuJoCo capsules joined by ball joints (rotation only). DER node forces $\vec{F}_j$ are converted to joint torques $\vec{T}_i = \sum_j \vec{F}_j \times \vec{D}_{i,j}$ (with sign chosen for the parent→child kinematic-tree orientation MuJoCo requires) and applied via `qfrc_passive`. Inextensibility is enforced by MuJoCo's constraint solver; damping is the standard MuJoCo joint damping.
- **Quasi-static centerline twist.** The material-frame twist is solved as the energy-minimizing twist for the current centerline at every step, removing the dynamic twist-wave artifact that plagues `native`. This both stabilizes simulation and removes the need for axial-rotation damping (which MuJoCo cannot specify per-axis on ball joints).
- **Parameter identification pipeline.**
  1. **Bending test.** Fix one end of a 1.5 m DLO in a 3D-printed custom twist apparatus (CTA), displace the other end 10 cm at 45°, release, capture the equilibrium with an Azure Kinect over 50 nodes; golden-section search on $\alpha$ minimizes 2-norm node-position error against the matching simulated equilibrium.
  2. **Twist test.** Same dangling-loop CTA setup; introduce twist in 5° increments until critical-twist buckling occurs in the real loop; bisect on $\beta/\alpha$ to match the real critical angle in simulation.
  3. **Cloud-size consistency metric.** Repeat $M=5$ times, report normalized parameter-cloud size $S = \frac{1}{M}\sum_i |\bar p - p_i|/\bar p$ as a per-DLO suitability score.
- **Tests.**
  - Two analytical-validation tests from the DER literature: **localized helical buckling** (Van der Heijden / Thompson) and **Michell's buckling instability** (critical twist for an elastic ring).
  - Real-vs-sim shape comparison: 0.40 m DLO held by a Denso VS-060 robot in 4 representative poses with axial-rotation twist at the fixed end; three DLO materials (white silicone, black PVC w/ copper, red nylon-braided PVC w/ copper); 10 discrete sections; normalized 2-norm position error per node.

## Results

- **Computation.** With $n$ from 40 to 180, `adapted` adds only $-1\%$ to $+3\%$ wall-time overhead over `plain` (no stiffness), vs. $+2\%$ to $+15\%$ for direct DER application; `native` is $-3\%$ to $+2\%$. Time step $\Delta t = 1.5\,\mathrm{ms}$.
- **Localized helical buckling (LHB).** Average 2-norm error vs. analytical envelope, decreasing $n$ ladder $\{40, 60, 80, 110, 140, 180\}$:
  - `adapted`: $0.0257 \to 0.0141 \to 0.00900 \to 0.00493 \to 0.00315 \to 0.00189$
  - `native`:  $0.0389 \to 0.0624 \to 0.00692 \to 0.00673 \to 0.00397 \to 0.00348$
  - `adapted` converges monotonically; `native` is non-monotone and consistently higher at refined $n$.
- **Michell's buckling instability.** Average 2-norm error in critical-twist-angle vs. $\beta/\alpha$ curve: `adapted` $= 0.0483$ vs. `native` $= 0.7725$ (≈16× worse). `native` produces a spurious roughly-linear $\theta^n_c \propto \beta/\alpha$ relation, inconsistent with Goriely's analytical curve.
- **Parameter cloud size $S$ over 5 repeats** (white / black / red): $S_{\text{bend}} = 2.18\% / 6.06\% / 7.55\%$; $S_{\text{twist}} = 0.16\% / 1.62\% / 2.30\%$. Order tracks the (qualitatively assessed) plastic-deformation severity of the three DLOs.
- **Real-vs-sim 4-pose shape comparison.** `adapted` produces lower normalized position error than `native` across all 3 DLOs and 4 poses; per-node error stays below 5% of total length without any real-data fine-tuning, **using only the parameters from the identification pipeline**. Two cases (black pose 2, red pose 4) show large `native` spikes attributable to its unnatural twist-wave oscillations; `adapted` does not exhibit them.

## Limitations

- **Static / quasi-dynamic regime only.** Validation focuses on equilibrium shapes and quasi-static twist transitions, not on dynamic flicking, throwing, or contact-rich manipulation. The transferability of the accuracy gains to fast-tip-velocity tasks is not demonstrated in this paper.
- **Inextensibility assumed.** Stretching and shearing are dropped. Plausible for steel cable / suture / wire harness; less so for highly elastic ropes.
- **Plastic deformation ignored.** Parameter cloud size grows monotonically with the DLOs' plastic behavior, so the identification is least reliable on the most plastic samples (red).
- **No batched / GPU rollout.** The implementation is a CPU MuJoCo plugin, sequential per environment. The paper itself is not differentiable; if the DER step is wrapped in MJX it could become trivially batched, but this is not done here.
- **Manual DLO segmentation.** Real-experiment node detection uses green markers + manual run of the DLO-detection routine, not an automated perception pipeline.
- **No closed-loop control validation.** No policy or planner is trained or evaluated on top of the simulator; the paper's claim is on simulation accuracy and parameter identification, not downstream task performance.

## Open questions

- How much of the static accuracy advantage transfers to **dynamic** regimes (whipping, swinging, throwing) where the linear-stiffness `native` model would similarly miss curvature-twist coupling?
- Does the generalized-coordinate DER step compose cleanly with **MJX**'s vectorized GPU pipeline to enable batched RL rollouts at MuJoCo time scales? (Compatible in principle — same physics — but throughput at scale is unmeasured.)
- Can a **learning-augmented** layer on top of identified $\alpha, \beta$ capture residual hysteresis / plastic effects without losing the warm-start advantage the paper emphasizes?
- Could the same force-lever conversion be applied to higher-order rod models (Cosserat with stretch and shear) without re-introducing the Jacobian-query overhead?

## My take

The contribution that matters for the DeformY arc is the **clean architectural bridge**: DER's bending+twisting energies, including the quasi-static twist treatment, expressed as joint torques via force-lever and dropped into MuJoCo's existing constraint+ball-joint machinery. That bridge is engineered to keep MuJoCo's API surface unchanged — there is no second simulator, no multi-rate co-simulation, no new contact code — which makes it a far cheaper substrate to adopt than e.g. dedicated co-simulation patterns like [[cosserat-isaac-cosimulation]] or differentiable stand-alone DER engines like [[deform-differentiable-discrete-elastic-rods-real]]. The price is: it is a CPU plugin, not differentiable, and the paper validates only static / quasi-static accuracy. For dynamic tip-targeting (the DeformY problem), this is the natural baseline simulator to compare any closed-loop policy against — accurate where DER theory says it should be, embedded in a robot-learning sim, and reproducible from the open-source repo. The parameter-identification pipeline (depth camera + 3D-printed CTA + golden-section / bisection on two scalar tests) is unusually low-friction and is the natural way to seed a real-DLO sim2real warm start.

## Related

**Foundations used**
- [[discrete-elastic-rods]] — the bending+twisting energy formulation adapted into MuJoCo
- [[cosserat-rod-theory]] — the broader rod theory DER discretizes (cited as the parent theory; stretch/shear dropped here)
- [[mass-spring-system]] — discussed and rejected as the alternative DLO representation
- [[position-based-dynamics]] — discussed as visually-plausible alternative
- [[deformable-linear-object]] — object class
- [[sim-to-real-transfer]] — claimed motivation: warm-start for sim2real DLO learning

**Concepts introduced**
- [[der-mujoco-generalized-coordinate-coupling]] — force-lever conversion of DER Cartesian centerline forces to MuJoCo ball-joint torques, enabling DER physics inside a generalized-coordinate solver

**Claims supported**
- [[der-mujoco-improves-static-dlo-accuracy-over-native-cable]]

**Key authors**
- [[qi-jing-chen]]

**Important referenced work** (sibling papers, may resolve at fan-in)
- Bergou et al. (2008), "Discrete Elastic Rods" — origin of the DER formulation
- Iterative Residual Policy (Chi et al., arXiv:2203.00663) — example of DLO learning workflow that benefits from accurate sim
- Robots of the Lost Arc (Zhang et al., 2021) — example of self-supervised DLO learning that the parameter pipeline could warm-start
- MuJoCo (Todorov et al., 2012) — host simulator
