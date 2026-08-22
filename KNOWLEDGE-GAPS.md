# Knowledge Gaps

This file tracks the learner's current understanding. It should be updated as misconceptions are corrected and concepts become reliable.

> Status is evidence-based. Recent topic-specific notes and Socratic reasoning take precedence over older checkboxes or calendar dates.

## Reliable / Working Understanding

### Background and estimation intuition

- [x] Probability and statistics
- [x] Gaussian distributions and uncertainty intuition
- [x] Bayesian inference intuition
- [x] Basic machine-learning background
- [x] Sensor confidence ↔ covariance / information intuition
- [x] Basic Kalman-filter concept
- [x] Why IMU integration drifts
- [x] Constant accelerometer bias can produce approximately (t^2) position error
- [x] Localization and mapping are coupled
- [x] State/process model vs measurement model distinction
- [x] PX4 Offboard autonomous-flight experience

### Geometry and SLAM optimization

- [x] Coordinate-frame direction and notation in SE(2), including `T_AB` as B → A
- [x] 2D rigid-body transformations
- [x] SO(2) and SE(2)
- [x] Transform composition, inverse, and relative pose
- [x] Point vs direction-vector transformation
- [x] Odometry as a relative-pose constraint
- [x] Pose graphs and factor graphs
- [x] Residuals, Gaussian factors, covariance, and information weighting
- [x] MLE / MAP connection to weighted nonlinear least squares
- [x] Local linearization and Jacobian as residual sensitivity
- [x] Gauss-Newton conceptual structure
- [x] Factor connectivity → sparse Jacobian / Hessian
- [x] Variable elimination and fill-in
- [x] Schur complement
- [x] Landmark elimination in Bundle Adjustment
- [x] Marginalization and marginalization priors
- [x] Sliding-window VIO motivation and bounded-computation trade-off
- [x] Distinction between a local estimator and a global pose-graph backend
- [x] Gauge freedom and gauge fixing at a conceptual level

## Partially Known / Needs Reinforcement

- [ ] Ground-robot differential-drive kinematics — current frontier; turning direction and wheelbase intuition are present, but exact (v,omega) relationships and angular-velocity meaning need consolidation
- [ ] Wheel odometry integration into an SE(2) pose
- [ ] Wheel-odometry assumptions, covariance, systematic errors, and slip failure modes
- [ ] SO(3) / SE(3) generalization from the established SO(2) / SE(2) model
- [ ] EKF details
- [ ] ESKF details
- [ ] Levenberg-Marquardt
- [ ] Visual-odometry scale ambiguity — central issue understood, but full geometry is pending
- [ ] VIO architecture — sliding-window marginalization is understood, but measurement models, bias, initialization, and preintegration are pending
- [ ] SLAM architecture — factor-graph backend and local/global distinction are understood, but a complete sensor-to-map implementation model needs reinforcement
- [ ] Loop closure — optimization role understood; place recognition, verification, and practical failure handling remain pending

## Critical Unresolved Gaps

- [ ] Differential-drive and other ground-robot motion models
- [ ] Nonholonomic constraints and when they fail
- [ ] Wheel encoder measurement model and calibration
- [ ] Wheel slip and odometry error models
- [ ] SO(3) rotation mathematics
- [ ] SE(3) transformations
- [ ] Robotics perturbation conventions and manifold updates
- [ ] EKF / ESKF implementation-level reasoning
- [ ] IMU measurement model and bias estimation
- [ ] LiDAR geometry
- [ ] Scan matching and ICP
- [ ] LiDAR odometry
- [ ] Camera model and calibration
- [ ] Feature geometry and data association
- [ ] Epipolar geometry
- [ ] Triangulation and visual odometry
- [ ] IMU preintegration
- [ ] VIO / LIO initialization and observability
- [ ] Practical front-end architecture
- [ ] Place recognition and loop-closure verification
- [ ] ROS 2 SLAM frames, timing, transforms, and stack integration
- [ ] Ground-robot sensor and SLAM-stack selection
- [ ] Failure analysis, experiment design, and evaluation metrics
- [ ] Ground-robot SLAM system design

## Deferred Until Justified

These are not immediate gaps merely because they are advanced topics. Study them when a practical implementation need or explicit prerequisite chain appears.

- [ ] Bayes Tree
- [ ] iSAM2
- [ ] FEJ
- [ ] Detailed elimination ordering

## Corrected Misconceptions

### Monocular visual odometry

Initial belief: camera-only localization is effectively impossible because feature measurements contain errors.

Correction: monocular VO/SLAM is possible. A central limitation is metric-scale ambiguity and other observability issues.

### Rotation inverse

Initial belief: inverse should exist because a transformation needs to be one-to-one.

Correction: for a rotation matrix, `R^-1 = R^T` because (R) is orthonormal.

### IMU and process model

Initial belief: IMU is simply a process-model sensor.

Correction: IMU produces measurements; estimators commonly use those measurements as inputs to high-rate state propagation.

### Odometry error growth

Initial belief: odometry error grows as (t^2) in general.

Correction: a constant acceleration bias gives a (t^2) position-error term. Different error sources have different growth characteristics.

## Current Focus

```text
Differential-drive wheel speeds
→ robot-frame linear and angular velocity
→ SE(2) pose integration
→ wheel-odometry assumptions and failure modes
→ probabilistic motion model / estimator connection
```
