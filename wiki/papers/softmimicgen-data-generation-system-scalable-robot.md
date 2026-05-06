---
title: "SoftMimicGen: A Data Generation System for Scalable Robot Learning in Deformable Object Manipulation"
slug: "softmimicgen-data-generation-system-scalable-robot"
arxiv: "2603.25725"
venue: "arXiv preprint"
year: 2026
tags: [deformable, data-generation, imitation-learning, mimicgen, non-rigid-registration, isaac-lab, sim-to-real, dynamic-manipulation, whipping, threading, folding, humanoid, surgical, bimanual]
importance: 3
date_added: 2026-05-06
source_type: tex
s2_id: "c4da6552dddc5841e86a978bb166b261d15939fa"
keywords: [SoftMimicGen, MimicGen, non-rigid registration, TPS, deformable objects, automated data generation, imitation learning, BC-RNN-GMM, Diffusion Policy, Isaac Lab, sim-real co-training, whipping, threading, folding, towel, rope, tissue]
domain: "Robotics"
code_url: "https://softmimicgen.github.io"
cited_by: []
---

## Problem

Synthetic data generation pipelines that bootstrap large imitation-learning datasets from a handful of human teleoperated demos (e.g. MimicGen, DexMimicGen, SkillMimicGen) have unlocked rigid-body manipulation at scale, but they all rest on the same load-bearing assumption: a static object reference frame exists, and demonstrations can be transferred via a constant SE(3) transform. Deformable objects break that assumption — rope, cloth, tissue, and stuffed bodies have continuous, high-dimensional configurations with no canonical frame, so the rigid-transfer trick fails immediately and rigid-body MimicGen succeeds only on configurations almost identical to the source demo. As a result, deformable manipulation has remained stuck on the small-real-dataset side of the data-paradigm divide that has powered rigid manipulation foundation models.

## Key idea

Replace the constant-SE(3) trajectory transform of MimicGen with **non-rigid registration**: model the deformable object as a set of 3D node positions, fit a smooth deformation field $\mathbf{f}: \mathbb{R}^3 \to \mathbb{R}^3$ between the source-demo object configuration and the new-scene object configuration (Schulman-style TPS-RPM), use the registration cost to *select* the best source segment, and use the deformation field plus its Jacobian to *warp* every end-effector pose in the segment so the local end-effector–object spatial relationship is preserved as the object shape changes. This turns object-centric trajectory transfer into a strict generalization of MimicGen — rigid bodies are recovered as a degenerate case (constant Jacobian, no deformation).

## Method

- **Object representation (A0).** Every deformable object is a set of 3D node positions $O = \{\mathbf{n}_i\}_{i=1}^{N_O}$, $\mathbf{n}_i \in \mathbb{R}^3$, obtained from the soft-body solver in Isaac Lab. Equivalent to a point cloud; rigid bodies can be represented the same way.
- **Per-subtask non-rigid registration.** For each object-centric subtask $S_i$ at data-gen time, observe the new object configuration $O'_i$, run non-rigid registration (point-wise distance + smoothness regularizer; no a-priori correspondences) against each candidate source segment's start configuration $O_i$, and pick the segment with lowest registration cost — a deformable analog of MimicGen's pose-distance nearest-neighbor selection.
- **Trajectory adaptation.** Given the chosen segment $\tau_i = (T^{C_0}_W, ..., T^{C_K}_W)$ and the deformation field $\mathbf{f}$ produced by registration, transform each pose $T_t = (p_t, R_t)$ as
  $$p_t \to \mathbf{f}(p_t),\quad R_t \to \mathrm{orth}(\mathbf{J}_\mathbf{f}(p_t)\,R_t)$$
  where $\mathrm{orth}(\cdot)$ orthonormalizes the rotation and $\mathbf{J}_\mathbf{f}$ is the field's Jacobian. Add a linear interpolation prefix from the robot's current pose to the warped trajectory's start, execute, and keep the rollout if the success predicate holds.
- **Simulation suite.** Ten high-fidelity Isaac Lab environments spanning four embodiments (Franka, bimanual YAM, GR1 humanoid, surgical robot) and four object classes (stuffed animal, rope, tissue, towel) covering pick-and-place, towel folding, towel unfold, rope U-shaping, dynamic Jenga whipping, surgical tissue retraction, and surgical thread-through-ring threading.
- **Teleoperation.** Apple Vision Pro retargets human hand motions to either a parallel-jaw gripper or a dexterous hand depending on embodiment; bimanual YAM is controlled in joint space.
- **Policy training.** From 1–3 source demos per task, generate 1,000 demos per task via SoftMimicGen, then train BC-RNN-GMM and Diffusion Policy visuomotor policies. Three real-world settings: real-only (30 demos), zero-shot sim-to-real, and sim–real co-training (30 real + 1,000 synthetic). Real-world deployment uses Point Bridge for unified point-based observations across sim and real.

## Results

- **Generation success rates.** SoftMimicGen produces successful demonstrations at 70–100% across the ten-task suite, from a single human demo plus broader initial-state randomization.
- **Policies trained on generated data beat policies trained on source demos alone** — improvements of 25–97% on success rate (Table 1). All ten tasks are non-trivial; whipping, threading, and folding are the dynamic / high-precision cases.
- **Direct head-to-head against MimicGen (Franka — Rope U-shape, single source demo, 50 trials):** MimicGen 4/50 successes (only configurations close to the source demo), SoftMimicGen 49/50 — a ~12× improvement, because the constant-SE(3) transform cannot follow a deforming rope.
- **Dataset scaling.** Policy success increases monotonically with dataset size from 50 → 750 demos (Table 2), confirming that the bottleneck is data quantity rather than imitation-learning capacity.
- **Sim-to-real (Table 3, three real tasks).** Policies trained on SoftMimicGen-generated data alone achieve nontrivial zero-shot sim-to-real transfer; sim–real co-training with 30 real demos further improves over real-only 30-demo training.

## Limitations

- **Fixed object-centric subtask sequence.** SoftMimicGen inherits MimicGen's assumption that tasks decompose into a deterministic ordered list of object-centric subtasks. Many real deformable tasks (untangling, bag manipulation, grasp recovery) need conditional transitions, retries, or branching — not in scope.
- **Soft-body simulator dependence.** The pipeline uses ground-truth nodal positions from Isaac Lab's soft solver for registration. Real-world deployment relies on Point Bridge to extract task-relevant point clouds; performance will depend on sensor and segmentation quality.
- **Single-source registration cost.** TPS-style non-rigid registration is per-pair and per-subtask; cost scales with subtask count × source candidates × demos generated.
- **Open-loop trajectory replay.** The data generator itself is open-loop with a success predicate gate; no closed-loop reactive correction during generation.
- **Non-rigid registration assumes consistent topology.** Two configurations are compared as smooth warps; large topology changes (tearing, knot-untangling) are out of scope.
- **No formal scaling law / no novel architecture.** The contribution is the data-generation mechanism plus benchmark suite, not new policies. Most absolute numbers are from BC-RNN-GMM / Diffusion Policy with standard settings.

## Open questions

- **Does the SoftMimicGen-data → policy → sim-to-real pipeline survive when the source demo is sparse and noisy** (e.g. teleoperated through pure RGB rather than Vision Pro)? Real-world data collection is unlikely to come with hand-tracked precision.
- **Is non-rigid registration the right primitive for *dynamic* trajectory transfer**, or does it underweight time-varying object state? The current method warps each pose with the registration field at $t=0$ of the segment; whether the field should evolve along the segment is left open.
- **Can SoftMimicGen be combined with skill-level motion planning (SkillMimicGen)** to handle long-horizon deformable tasks with retries? The Related Work flags this as complementary, not integrated.
- **What is the irreducible real-world demo budget once SoftMimicGen + co-training is used?** The paper shows 30 real + 1,000 sim improves over 30 real, but does not isolate the diminishing-returns curve.
- **How does the registration-cost source selection criterion compare to learned source-selection** (e.g. a small policy that picks the source segment given current observations)?

## My take

This is the first paper to scale a MimicGen-style automated trajectory-transfer pipeline to *deformable* objects across multiple embodiments — and the Franka-Rope head-to-head against MimicGen (4/50 vs. 49/50) is the cleanest demonstration in the literature that the rigid SE(3)-transfer assumption is the actual bottleneck, not a missing detail of the imitation-learning stack. The choice of non-rigid registration (specifically Schulman-style TPS-RPM) as the trajectory-warp primitive is conservative: it is well-understood, has a long pedigree in deformable manipulation since Schulman 2013/2016, and degenerates cleanly to rigid transfer. The price is the assumption of a fixed object-centric subtask sequence — the paper acknowledges this is the binding constraint for real-world long-horizon deformable manipulation. For DeformY's purpose: SoftMimicGen is the strongest existing argument that *automated demo generation*, not *new policy architectures*, is the leveraged path for scaling deformable manipulation, and the dynamic-whipping benchmark (Franka–Jenga) is the closest published analog to a closed-loop dynamic-DLO task — though it is open-loop replay of warped expert trajectories rather than reactive control.

## Related

**Foundations used**
- [[behavioral-cloning]] — policies are trained via maximum-likelihood BC on the generated dataset.
- [[imitation-learning]] — the broader paradigm being scaled.
- [[diffusion-policy]] — one of the two visuomotor policy classes evaluated (the other is BC-RNN-GMM).
- [[visuomotor-policy]] — image-conditioned policies bypass explicit registration at inference time.
- [[deformable-linear-object]] — rope, threading, surgical thread.
- [[sim-to-real-transfer]] — zero-shot and co-training results in Table 3.
- [[contact-rich-manipulation]] — threading, whipping, folding all involve sustained contact.
- [[domain-randomization]] — broader initial-state distribution at gen time supplies an analog of policy-time DR.

**Concepts introduced**
- [[automated-demo-generation-deformable]] — the SoftMimicGen mechanism: non-rigid-registration-based source selection + Jacobian-warped trajectory adaptation as a strict generalization of object-centric MimicGen-style data generation, applicable to deformable and rigid manipulation alike.

**Claims supported**
- [[mimicgen-style-pipeline-scales-to-dynamic-deformables]]

**Important referenced work** (not yet ingested — candidates for follow-up `/ingest`)
- MimicGen (Mandlekar et al., 2023) — the rigid-body parent system whose SE(3)-transfer assumption SoftMimicGen relaxes.
- DexMimicGen (Jiang et al., 2024) — bimanual extension of MimicGen.
- SkillMimicGen (Garrett et al., 2024) — motion-planning extension of MimicGen, flagged as complementary.
- Schulman 2013 / 2016 — the TPS-RPM-style non-rigid-registration trajectory-transfer line that SoftMimicGen builds on.
- Diffusion Policy (Chi et al., 2023) — one of the two policy classes used.
- BC-RNN-GMM (Mandlekar et al., 2021) — the other policy class.
- Isaac Lab (Mittal et al., 2025) — simulation backend.
- Point Bridge (Haldar et al., 2026) — unified point-based representation for sim-to-real deployment.
- pi_0 / OpenVLA / GR00T / RT-2 — the foundation-model context the paper situates itself in.
