# Query Pack (general)

_Auto-generated compressed context. Do not edit._

## Claims (27 total)
- [weakly_supported] Free-end cable dynamic target reaching is harder than fixed-end on per-cable-length error (conf: 0.55)
- [supported] DaXBench Whip-Rope baselines: APG 0.83 dominates SHAC 0.66 and PPO 0.25 — analytic policy gradients win on dynamic deformable tasks (conf: 0.7)
- [weakly_supported] Cosserat physics narrows the DLO swinging sim-to-real gap (conf: 0.55)
- [weakly_supported] Decoupling sysID from task execution beats end-to-end iterative residual policies on the 3D rope-tip striking task at deployment time (conf: 0.5)
- [weakly_supported] Learning delta dynamics from observed trajectories generalizes better than explicit system identification for dynamic deformable manipulation (conf: 0.7)
- [weakly_supported] Integrating DER bending+twisting energies into MuJoCo's generalized-coordinate solver yields more accurate static / quasi-dynamic DLO simulation than MuJoCo's native cable plugin (conf: 0.7)
- [supported] DIDP reaches 84.3% within 5 cm and 20.8% within 1 cm tip-to-goal on a 3D reduced-order Cosserat rope-whipping benchmark (conf: 0.7)
- [supported] Differentiable DER plus a neural residual achieves real-time, accurate dynamic DLO modeling, perception, and shape-matching control on real robots (conf: 0.75)
- [supported] A learned 3D apex point that parameterizes a minimum-jerk QP trajectory is sufficient to learn target-conditioned dynamic cable behaviors (vaulting / knocking / weaving) on real hardware across multiple cables (conf: 0.7)
- [weakly_supported] Heterogeneous payload-on-rope dynamics are captured implicitly without per-trial residual updates, in contrast to IRP's iterative residual loop (conf: 0.55)
- [supported] 100 Hz nominal MPC + 400 Hz RL residual + pullback-tube optimizer enables 0.276 m mean landing error at 6 m on ANYmal-D + DynaArm (conf: 0.75)
- [weakly_supported] Implicit sysID encoder + goal-conditioned action predictor enables one-shot real-deployment for rope-as-tool 3D-target transport of rigid payloads (conf: 0.65)

## Open Gaps
_Auto-generated open questions. Do not edit._
- [paper/accurate-simulation-parameter-identification-dlos-using] How much of the static accuracy advantage transfers to **dynamic** regimes (whipping, swinging, throwing) where the linear-stiffness `native` model would similarly miss curvature-twist coupling?
- [paper/accurate-simulation-parameter-identification-dlos-using] Does the generalized-coordinate DER step compose cleanly with **MJX**'s vectorized GPU pipeline to enable batched RL rollouts at MuJoCo time scales? (Compatible in principle — same physics — but throughput at scale is unmeasured.)
- [paper/accurate-simulation-parameter-identification-dlos-using] Can a **learning-augmented** layer on top of identified $\alpha, \beta$ capture residual hysteresis / plastic effects without losing the warm-start advantage the paper emphasizes?
- [paper/accurate-simulation-parameter-identification-dlos-using] Could the same force-lever conversion be applied to higher-order rod models (Cosserat with stretch and shear) without re-introducing the Jacobian-query overhead?
- [paper/daxbench-benchmarking-deformable-object-manipulation-differentiable] How much of APG's Whip-Rope win is due to MLS-MPM particle physics vs. due to JAX-batched gradient flow? Would the same algorithm dominate on a Cosserat-rod or linked-capsule rope?
- [paper/daxbench-benchmarking-deformable-object-manipulation-differentiable] Can entropy-regularized differentiable RL (e.g. add policy entropy bonus to APG) clos
## Papers (18 total)
- [4] DEFORM: Differentiable Discrete Elastic Rods for Real-Time Modeling of Deformable Linear Objects (Robotics)
- [5] Iterative Residual Policy: For Goal-Conditioned Dynamic Manipulation of Deformable Objects (Robotics)
- [4] Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy (Robotics)
- [4] Real2Sim2Real: Self-Supervised Learning of Physical Single-Step Dynamic Actions for Planar Robot Casting (Robotics)
- [4] Self-Supervised Learning of Dynamic Planar Manipulation of Free-End Cables (Robotics)
- [4] Learning Accurate Whole-body Throwing with High-frequency Residual Policy and Pullback Tube Acceleration (Robotics)
- [3] SoftMimicGen: A Data Generation System for Scalable Robot Learning in Deformable Object Manipulation (Robotics)
- [4] DaXBench: Benchmarking Deformable Object Manipulation with Differentiable Physics (Robotics)
- [4] Wiggle and Go! System Identification for Zero-Shot Dynamic Rope Manipulation (Robotics)
- [4] Learning Deformable Object Manipulation Using Task-Level Iterative Learning Control (Robotics)
- [5] TossingBot: Learning to Throw Arbitrary Objects with Residual Physics (Robotics)
- [4] Accurate Simulation and Parameter Identification of Deformable Linear Objects using Discrete Elastic Rods in Generalized Coordinates (Robotics)
- [3] Rapid Adaptation of Particle Dynamics for Generalized Deformable Object Mobile Manipulation (Robotics)
- [4] Robots of the Lost Arc: Self-Supervised Learning to Dynamically Manipulate Fixed-Endpoint Cables (Robotics)
- [3] Self-Curriculum Model-based Reinforcement Learning for Shape Control of Deformable Linear Objects (Robotics)
## Recent Relationships (181 total)
  papers/ropedreamer-kinematic-recurrent-state-space-model --derived_from--> foundations/forward-kinematics
  papers/ropedreamer-kinematic-recurrent-state-space-model --derived_from--> foundations/model-based-reinforcement-learning
  papers/ropedreamer-kinematic-recurrent-state-space-model --introduces_concept--> concepts/quaternionic-rssm-dlo
  papers/ropedreamer-kinematic-recurrent-state-space-model --supports--> claims/quaternionic-kinematic-rssm-reduces-dlo-rollout-error
  papers/rapid-adaptation-particle-dynamics-generalized-deformable --introduces_concept--> concepts/rma-particle-dynamics-adaptation
  papers/rapid-adaptation-particle-dynamics-generalized-deformable --supports--> claims/rma-particle-rapid-real-world-success
  papers/rapid-adaptation-particle-dynamics-generalized-deformable --derived_from--> foundations/deformable-linear-object
  papers/rapid-adaptation-particle-dynamics-generalized-deformable --derived_from--> foundations/sim-to-real-transfer
  papers/rapid-adapta
