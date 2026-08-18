# Knowledge Gaps

This file tracks the learner's current understanding. It should be updated as misconceptions are corrected and concepts become reliable.

## Strong / Existing Foundation

- [x] Probability and statistics
- [x] Gaussian distributions and uncertainty intuition
- [x] Bayesian inference intuition
- [x] Basic machine-learning background
- [x] Sensor confidence ↔ covariance intuition
- [x] Basic Kalman filter concept
- [x] PX4 Offboard autonomous-flight experience

## Partially Known

- [x] Why IMU integration drifts
- [x] Constant accelerometer bias can produce approximately t^2 position error
- [x] Localization and mapping are coupled
- [x] State/process model vs measurement model distinction
- [x] Basic robot/world coordinate transformation intuition
- [ ] EKF details
- [ ] ESKF details

## Critical Gaps

- [ ] Coordinate-frame conventions and notation
- [ ] Rigid-body transformations
- [ ] SO(2)/SO(3)
- [ ] SE(2)/SE(3)
- [ ] Relative pose and transform composition
- [ ] Ground robot kinematics
- [ ] Wheel odometry error models
- [ ] Jacobians and linearization in robotics
- [ ] Nonlinear least squares
- [ ] Gauss-Newton / Levenberg-Marquardt
- [ ] Factor graphs
- [ ] Pose graphs
- [ ] LiDAR scan matching
- [ ] ICP
- [ ] Visual geometry
- [ ] Epipolar geometry
- [ ] Visual odometry
- [ ] Monocular scale ambiguity
- [ ] IMU measurement model and bias estimation
- [ ] VIO
- [ ] IMU preintegration
- [ ] SLAM front-end/back-end architecture
- [ ] Loop closure
- [ ] Ground-robot SLAM system design

## Corrected Misconceptions

### Monocular visual odometry
Initial belief: camera-only localization is effectively impossible because feature measurements contain errors.

Correction: monocular VO/SLAM is possible. A central limitation is metric-scale ambiguity and other observability issues.

### Rotation inverse
Initial belief: inverse should exist because a transformation needs to be one-to-one.

Correction: for a rotation matrix, `R^-1 = R^T` because R is orthonormal.

### IMU and process model
Initial belief: IMU is simply a process-model sensor.

Correction: IMU produces measurements; estimators commonly use those measurements as inputs to high-rate state propagation.

### Odometry error growth
Initial belief: odometry error grows as t^2 in general.

Correction: a constant acceleration bias gives a t^2 position-error term. Different error sources have different growth characteristics.
