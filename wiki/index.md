# Wiki Index

papers:
  - slug: deformx-versatile-co-simulation-framework-deformable
    title: "DeformX: A Versatile Co-Simulation Framework for Deformable Linear Objects"
    tags: [DLO, deformable-linear-object, cosserat-rod, simulation, sim-to-real, robot-learning, dataset, isaac-sim]
    importance: 3
    domain: Robotics
  - slug: dynamic-manipulation-deformable-objects-3d-simulation
    title: "Dynamic Manipulation of Deformable Objects in 3D: Simulation, Benchmark and Learning Strategy"
    tags: [DLO, deformable-linear-object, diffusion-policy, test-time-adaptation, reduced-order-model, GVS, cosserat-rod, benchmark, 3D-rope-manipulation, simulation, sim-only]
    importance: 4
    domain: Robotics

concepts:
  - slug: cosserat-isaac-cosimulation
    title: "Cosserat-Isaac Co-Simulation"
    tags: [DLO, simulation, cosserat-rod, isaac-sim, sim-to-real, robot-learning]
    maturity: emerging
  - slug: dynamics-informed-diffusion-policy
    title: "Dynamics-Informed Diffusion Policy (DIDP)"
    tags: [DLO, diffusion-policy, imitation-learning, trajectory-optimization, reduced-order-model, robot-learning]
    maturity: emerging
  - slug: physics-informed-test-time-adaptation
    title: "Physics-Informed Test-Time Adaptation (PITA) for Diffusion Sampling"
    tags: [diffusion-policy, test-time-adaptation, score-based-sampling, physics-prior, robot-learning, DLO]
    maturity: emerging
  - slug: reduced-order-gvs-model
    title: "Reduced-Order GVS (Geometric Variable Strain) Model"
    tags: [DLO, cosserat-rod, simulation, reduced-order-model, differentiable-simulation, soft-robotics]
    maturity: emerging

topics:

people:
  - slug: guanzhou-lan
    title: "Guanzhou Lan"
    affiliation: "Northwestern Polytechnical University"

ideas:

experiments:

claims:
  - slug: cosserat-physics-narrows-dlo-swinging-sim2real
    title: "Cosserat physics narrows the DLO swinging sim-to-real gap"
    tags: [DLO, sim-to-real, cosserat-rod, robot-learning, PPO]
    status: weakly_supported
    confidence: 0.55
    domain: Robotics
  - slug: didp-3d-rope-tip-targeting-success-rates
    title: "DIDP reaches 84.3% within 5 cm and 20.8% within 1 cm tip-to-goal on a 3D reduced-order Cosserat rope-whipping benchmark"
    tags: [DLO, diffusion-policy, benchmark, success-rate, sim-only, 3D-rope-manipulation, GVS]
    status: supported
    confidence: 0.7
    domain: Robotics
  - slug: physics-informed-tta-improves-diffusion-policy-on-dynamic-dlo
    title: "Physics-informed test-time adaptation during diffusion sampling materially improves over plain diffusion policy on dynamic DLO tasks"
    tags: [diffusion-policy, test-time-adaptation, physics-prior, DLO, sim-only, robot-learning]
    status: weakly_supported
    confidence: 0.6
    domain: Robotics

Summary:

foundations:
  - slug: backpropagation
    title: "Backpropagation"
    status: mainstream
    domain: general
  - slug: behavioral-cloning
    title: "Behavioral Cloning"
    status: mainstream
    domain: Robotics
  - slug: contact-rich-manipulation
    title: "Contact-Rich Manipulation"
    status: mainstream
    domain: Robotics
  - slug: cosserat-rod-theory
    title: "Cosserat Rod Theory"
    status: mainstream
    domain: Robotics
  - slug: cross-validation
    title: "Cross-Validation"
    status: mainstream
    domain: general
  - slug: deformable-linear-object
    title: "Deformable Linear Object"
    status: mainstream
    domain: Robotics
  - slug: diffusion-policy
    title: "Diffusion Policy"
    status: mainstream
    domain: Robotics
  - slug: discrete-elastic-rods
    title: "Discrete Elastic Rods"
    status: mainstream
    domain: Robotics
  - slug: domain-randomization
    title: "Domain Randomization"
    status: mainstream
    domain: Robotics
  - slug: finite-element-method
    title: "Finite Element Method"
    status: mainstream
    domain: Robotics
  - slug: forward-kinematics
    title: "Forward Kinematics"
    status: mainstream
    domain: Robotics
  - slug: gradient-descent
    title: "Gradient Descent"
    status: mainstream
    domain: general
  - slug: grasping
    title: "Grasping"
    status: mainstream
    domain: Robotics
  - slug: imitation-learning
    title: "Imitation Learning"
    status: mainstream
    domain: Robotics
  - slug: impedance-control
    title: "Impedance Control"
    status: mainstream
    domain: Robotics
  - slug: inverse-kinematics
    title: "Inverse Kinematics"
    status: mainstream
    domain: Robotics
  - slug: jacobian-based-control
    title: "Jacobian-Based Control"
    status: mainstream
    domain: Robotics
  - slug: mass-spring-system
    title: "Mass-Spring System"
    status: mainstream
    domain: Robotics
  - slug: model-based-reinforcement-learning
    title: "Model-Based Reinforcement Learning"
    status: mainstream
    domain: Robotics
  - slug: model-predictive-control
    title: "Model Predictive Control"
    status: mainstream
    domain: Robotics
  - slug: optimization
    title: "Optimization"
    status: mainstream
    domain: general
  - slug: position-based-dynamics
    title: "Position-Based Dynamics"
    status: mainstream
    domain: Robotics
  - slug: regularization
    title: "Regularization"
    status: mainstream
    domain: general
  - slug: shape-servoing
    title: "Shape Servoing"
    status: mainstream
    domain: Robotics
  - slug: sim-to-real-transfer
    title: "Sim-to-Real Transfer"
    status: mainstream
    domain: Robotics
  - slug: tactile-sensing
    title: "Tactile Sensing"
    status: mainstream
    domain: Robotics
  - slug: visual-servoing
    title: "Visual Servoing"
    status: mainstream
    domain: Robotics
  - slug: visuomotor-policy
    title: "Visuomotor Policy"
    status: mainstream
    domain: Robotics
