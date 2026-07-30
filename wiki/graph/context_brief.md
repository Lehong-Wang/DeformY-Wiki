# Query Pack (general)

_Auto-generated compressed context. Do not edit._

## Methods (5 total)
- [inference] Diffuser: Guided Diffusion Planning
- [training] GrBAL / ReBAL: Online Meta-Learned Dynamics Adaptation
- [architecture] MMP++ / IMMP++ (Parametric-Curve Motion Manifold Primitives)
- [inference] PETS: Probabilistic Ensembles with Trajectory Sampling
- [training] RMA: Rapid Motor Adaptation (two-phase context-encoder adaptation)
## Open Gaps
_Auto-generated open questions. Do not edit._
- [paper/accurate-simulation-parameter-identification-dlos-using] How much of the static accuracy advantage transfers to **dynamic** regimes (whipping, swinging, throwing) where the linear-stiffness `native` model would similarly miss curvature-twist coupling?
- [paper/accurate-simulation-parameter-identification-dlos-using] Does the generalized-coordinate DER step compose cleanly with **MJX**'s vectorized GPU pipeline to enable batched RL rollouts at MuJoCo time scales? (Compatible in principle — same physics — but throughput at scale is unmeasured.)
- [paper/accurate-simulation-parameter-identification-dlos-using] Can a **learning-augmented** layer on top of identified $\alpha, \beta$ capture residual hysteresis / plastic effects without losing the warm-start advantage the paper emphasizes?
- [paper/accurate-simulation-parameter-identification-dlos-using] Could the same force-lever conversion be applied to higher-order rod models (Cosserat with stretch and shear) without re-introducing the Jacobian-query overhead?
- [paper/daxbench-benchmarking-deformable-object-manipulation-differentiable] How much of APG's Whip-Rope win is due to MLS-MPM particle physics vs. due to JAX-batched gradient flow? Would the same algorithm dominate on a Cosserat-rod or linked-capsule rope?
- [paper/daxbench-benchmarking-deformable-object-manipulation-differentiable] Can entropy-regularized differentiable RL (e.g. add policy entropy bonus to APG) clos
## Papers (23 total)
- [4] DEFORM: Differentiable Discrete Elastic Rods for Real-Time Modeling of Deformable Linear Objects
- [5] Iterative Residual Policy: For Goal-Conditioned Dynamic Manipulation of Deformable Objects
- [4] Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy
- [4] Learning to Adapt in Dynamic, Real-World Environments Through Meta-Reinforcement Learning — Meta-trains a dynamics-model prior whose update rule rapidly re-fits the model from the last M timesteps to predict the next K, giving model-based RL agents (GrBAL/ReBAL) fast online adaptation to crippled limbs, new terrains, and payloads — demonstrated on the first meta-RL real robot.
- [4] Real2Sim2Real: Self-Supervised Learning of Physical Single-Step Dynamic Actions for Planar Robot Casting
- [4] Wiggle and Go! System Identification for Zero-Shot Dynamic Rope Manipulation
- [4] Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration
- [4] Self-Supervised Learning of Dynamic Planar Manipulation of Free-End Cables
- [4] DaXBench: Benchmarking Deformable Object Manipulation with Differentiable Physics
- [5] Deep Reinforcement Learning in a Handful of Trials using Probabilistic Dynamics Models — PETS combines a probabilistic ensemble of bootstrapped neural dynamics models (separating aleatoric from epistemic uncertainty) with trajectory-sampling uncertainty propagation and CEM-based MPC planning, matching model-free asymptotic returns on MuJoCo control with 8-125x fewer samples.
- [3] SoftMimicGen: A Data Generation System for Scalable Robot Learning in Deformable Object Manipulation
- [3] Rapid Adaptation of Particle Dynamics for Generalized Deformable Object Mobile Manipulation
- [3] Self-Curriculum Model-based Reinforcement Learning for Shape Control of Deformable Linear Objects
- [5] TossingBot: Learning to Throw Arbitrary Objects with Residual Physics
- [4] Learning Deformable Object Manipulation Using Task-Level Iterativ
## Recent Relationships (227 total)
  papers/deep-reinforcement-learning-handful-trials-using --similar_method_to--> papers/wiggle-go-system-identification-zero-shot
  papers/deep-reinforcement-learning-handful-trials-using --similar_method_to--> papers/self-curriculum-model-based-reinforcement-learning
  papers/deep-reinforcement-learning-handful-trials-using --similar_method_to--> papers/learning-adapt-dynamic-real-world-environments
  papers/rapid-adaptation-particle-dynamics-generalized-deformable --builds_on--> papers/rma-rapid-motor-adaptation-legged-robots
  papers/learning-adapt-dynamic-real-world-environments --similar_method_to--> papers/rma-rapid-motor-adaptation-legged-robots
  papers/rma-rapid-motor-adaptation-legged-robots --similar_method_to--> papers/wiggle-go-system-identification-zero-shot
  papers/implicit-physics-aware-policy-dynamic-manipulation --similar_method_to--> papers/rma-rapid-motor-adaptation-legged-robots
  papers/rma-rapid-motor-adaptation-legged-robots --introduces_concept--> concepts/amo
