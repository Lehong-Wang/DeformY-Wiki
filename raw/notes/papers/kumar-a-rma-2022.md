---
title: "Adapting Rapid Motor Adaptation for Bipedal Robots"
authors: ["Ashish Kumar", "Zhongyu Li", "Jun Zeng", "Deepak Pathak", "Koushil Sreenath", "Jitendra Malik"]
venue: "IROS 2022"
year: 2022
arxiv_id: "2205.15299"
doi: "10.1109/IROS47612.2022.9981091"
note_type: bibliography_only
sources: [field-research]
---

# A-RMA: Adapting Rapid Motor Adaptation for Bipedal Robots

**One-line gist**: A two-phase meta-RL framework that adds a second adaptation stage — finetuning the base policy against an imperfect context encoder — to achieve zero-shot sim-to-real transfer on the unstable bipedal robot Cassie.

**Task/Method setup**: Bipedal locomotion (Cassie) over varied terrains and dynamics. Phase 1 (standard RMA): train base policy + extrinsics encoder in sim. Phase 2 (the "A" in A-RMA): freeze the encoder, then finetune the base policy via model-free RL using the (imperfect) encoder's output as input — explicitly hardening the policy against encoder errors.

**Sim vs real**: Trained entirely in simulation (GPU-parallel Isaac Gym); deployed zero-shot to real Cassie hardware. One-time real calibration not required per target; the encoder runs online from proprioception history.

**Core idea / mechanism**: RMA's original assumption is that the extrinsics encoder is accurate enough that the base policy can treat it as ground truth. A-RMA relaxes this: it fines-tunes the base policy *conditioned on the encoder's (noisy) output*, so the policy learns to be robust to encoder imperfection. The two-phase training decouples representation learning from robustness to representation error.

**Why it matters for OUR problem**:
- **Meta-adaptation / forward model**: A-RMA's context encoder (trained on proprioception history) is directly analogous to the RMA-style context encoder we plan to use for one-time real rope calibration — it collapses a short real interaction window into a latent that specialises the forward model without per-target re-training.
- **Sim2real**: The two-phase approach (sim prior → finetune base policy against imperfect encoder) is a concrete recipe for our sim-trained forward model that must specialise to a real rope's dynamics; phase 2 acts as the "calibration" step.
- **Robust planning**: Finetuning the policy against an imperfect encoder mirrors our need to plan robustly against model uncertainty — both guard against model-exploitation.
- **Compact action**: Not directly relevant, but the base policy in A-RMA outputs smooth joint targets, showing that a compact policy output is compatible with the RMA adaptation framework.

**Key result**: A-RMA outperforms RL baselines and model-based controllers in sim, and achieves zero-shot deployment to real Cassie across terrain, payload, and slope variations — demonstrating that the two-phase approach closes the remaining sim-to-real gap that vanilla RMA leaves open for bipedal systems.
