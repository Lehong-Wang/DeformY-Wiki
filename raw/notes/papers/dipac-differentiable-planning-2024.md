---
title: "DiPac: Differentiable Particles for General-Purpose Deformable Object Manipulation"
authors: [Siwei Chen, Yiqing Xu, Cunjun Yu, Linfeng Li, David Hsu]
venue: "arXiv preprint / ICRA 2024 Workshop on Representing and Manipulating Deformable Objects (invited talk)"
year: 2024
arxiv_id: "2405.01044"
doi: null
note_type: bibliography_only
sources: [field-research]
---

# DiPac — Differentiable Particles for DLO Manipulation

**One-line gist**: Represent deformable objects as particles, wrap them in a differentiable simulator, and do trajectory-tree optimization via gradient backprop to jointly estimate dynamics parameters and plan manipulation actions.

**Task/Method setup**: General-purpose manipulation of diverse deformable objects (rope, cloth, beans, liquid). A particle-based differentiable simulator models object dynamics. DiPac combines (1) learning (data-driven prior), (2) sampling-based planning (trajectory tree), and (3) gradient-based trajectory optimization — all unified through differentiable simulation. Dynamics parameters are estimated efficiently by backpropagating a mismatch loss.

**Sim vs real**: Full sim-to-real transfer. Differentiable parameter estimation (one-time calibration from a short real observation sequence) narrows the sim-to-real gap. Demonstrated on physical objects including rope.

**Core idea / mechanism**: Trajectory-tree optimization — sample a tree of candidate action sequences, evaluate each leaf via differentiable rollout, then backprop gradients through the tree to refine actions. Differentiable dynamics simultaneously solves system-ID (physics parameters) and action optimization in one pass. Enables policy transfer across objects with different physical properties (rigid rod → flexible rope) without re-training.

**Why it matters for OUR problem**:
- *Forward model*: DiPac's particle differentiable sim is directly applicable as a GPU-accelerated forward model for rope tip trajectory prediction; gradients are free.
- *Compact action / robust planning*: Trajectory-tree + gradient refinement is complementary to our spline-decoder + PETS-style planning — differentiable sim enables gradient-informed warm-starting of the ensemble planner, reducing the number of rollouts needed.
- *Sim2real / meta-adaptation*: Their differentiable parameter estimation from a short real sequence is structurally identical to our one-time per-rope calibration goal; the technique could replace or augment the RMA context-encoder's real-data fine-tune step.
- *Anti model-exploitation*: Tree-search + gradient descent within a differentiable model can still over-exploit; our ensemble pessimism is still needed on top, but DiPac's approach reduces gradient noise.
- *Wind-up*: Not addressed.

**Key result**: DiPac outperforms both pure model-based planners and pure data-driven policies across granular, cloth, rope, and liquid manipulation tasks; robustly transfers an expert policy from a rigid rod to a deformable rope with different physical parameters.
