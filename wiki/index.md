# Wiki Index

papers:
  - slug: deformx-versatile-co-simulation-framework-deformable
    title: "DeformX: A Versatile Co-Simulation Framework for Deformable Linear Objects"
    tags: [DLO, deformable-linear-object, cosserat-rod, simulation, sim-to-real, robot-learning, dataset, isaac-sim]
    importance: 3
    domain: Robotics
  - slug: learning-deformable-object-manipulation-using-task
    title: "Learning Deformable Object Manipulation Using Task-Level Iterative Learning Control"
    tags: [DLO, rope, dynamic-manipulation, iterative-learning-control, ILC, real-world-learning, flying-knot, knot-tying, xArm-7, model-based, single-demo, transfer, robot-learning]
    importance: 4
    domain: Robotics

concepts:
  - slug: cosserat-isaac-cosimulation
    title: "Cosserat-Isaac Co-Simulation"
    tags: [DLO, simulation, cosserat-rod, isaac-sim, sim-to-real, robot-learning]
    maturity: emerging
  - slug: task-level-iterative-learning-control
    title: "Task-Level Iterative Learning Control"
    tags: [iterative-learning-control, ILC, control, deformable-object-manipulation, robot-learning, real-world-learning]
    maturity: emerging
  - slug: critical-point-objective
    title: "Critical-Point Objective"
    tags: [iterative-learning-control, ILC, control, deformable-object-manipulation, trajectory-cost, behavior-cloning, robot-learning]
    maturity: emerging
  - slug: optimization-based-inverse-model
    title: "Optimization-Based Inverse Model (Norm-Optimal ILC)"
    tags: [iterative-learning-control, ILC, control, optimization, quadratic-program, robot-learning, inverse-model]
    maturity: active

topics:

people:
  - slug: krishna-suresh
    name: "Krishna Suresh"
    affiliation: "Carnegie Mellon University"
    tags: [robot-learning, iterative-learning-control, ILC, deformable-object-manipulation, rope-manipulation]

ideas:

experiments:

claims:
  - slug: cosserat-physics-narrows-dlo-swinging-sim2real
    title: "Cosserat physics narrows the DLO swinging sim-to-real gap"
    tags: [DLO, sim-to-real, cosserat-rod, robot-learning, PPO]
    status: weakly_supported
    confidence: 0.55
    domain: Robotics
  - slug: task-level-ilc-real-hardware-flying-knot-100pct-under-10-trials
    title: "Task-Level ILC from one human demo plus a 5-parameter rope model achieves 100% success in fewer than 10 real-hardware trials of the flying knot across 7 rope and cable types"
    tags: [ILC, iterative-learning-control, deformable-manipulation, rope, real-world-learning, sample-efficiency, xArm-7, flying-knot]
    status: supported
    confidence: 0.75
    domain: Robotics
  - slug: task-level-ilc-cross-rope-transfer-2-to-5-trials
    title: "Cross-rope command transfer with Task-Level ILC requires only 2-5 additional real trials for most rope-pair source/targets"
    tags: [ILC, iterative-learning-control, deformable-manipulation, rope, transfer-learning, real-world-learning, sample-efficiency]
    status: weakly_supported
    confidence: 0.65
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
