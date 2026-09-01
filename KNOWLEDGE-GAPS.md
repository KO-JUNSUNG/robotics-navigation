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
- [x] Constant accelerometer bias can produce approximately `t^2` position error
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
- [x] Differential-drive kinematics: wheel speeds, body linear velocity, yaw rate, and turning radius
- [x] Wheel-encoder increments to SE(2) pose integration using Euler / midpoint intuition
- [x] Nonholonomic lateral-velocity constraint and practical violations
- [x] Wheel slip, wheel-radius / wheelbase calibration errors, and multi-sensor disagreement
- [x] First-order motion-model Jacobian and covariance-propagation intuition
- [x] SO(3) matrix meaning, determinant, axis columns, inverse/transpose, and noncommutativity
- [x] SE(3) point transformation, composition, inverse, and frame-labeled extrinsics
- [x] LiDAR lever-arm effect and extrinsic-calibration failure intuition
- [x] Accelerometer specific force, stationary / free-fall distinction, and gravity-frame transformation
- [x] Gravity constrains roll/pitch but not yaw
- [x] Gyro bias accumulation and body-frame relative-rotation composition
- [x] ESKF nominal state versus small error state
- [x] SO(3)-valid orientation-error injection and error-mean reset
- [x] Error-state mean reset versus covariance preservation
- [x] Kalman-gain trust intuition and estimator overconfidence
- [x] Yaw-error / gyro-bias cross-covariance generation through propagation
- [x] Indirect gyro-bias correction through yaw-only measurement and cross-covariance
- [x] Cross-covariance versus observability distinction
- [x] Additive bias versus multiplicative scale-error excitation intuition
- [x] Measurement consistency, adaptive covariance, rejection, and duplicate-update failure intuition
- [x] Conceptual asynchronous IMU / wheel / LiDAR ESKF cycle
- [x] LiDAR scan frames, relative transform, and non-index correspondence
- [x] ICP local-optimization and initial-guess dependence
- [x] Point-to-line normal and corridor degeneracy intuition
- [x] Repeated-structure local minima versus continuous degeneracy
- [x] Per-point LiDAR timestamp and deskew motivation
- [x] LiDAR-relative to robot-relative pose extrinsic composition

## Partially Known / Needs Reinforcement

- [ ] EKF implementation details and comparison with ESKF
- [ ] ESKF propagation/update Jacobians, reset Jacobian, and implementation details
- [ ] Point-to-line ICP linearization, Jacobians, and implementation details
- [ ] Robust correspondence rejection and scan-matching quality metrics
- [ ] Levenberg-Marquardt
- [ ] Visual-odometry scale ambiguity — central issue understood, but full geometry is pending
- [ ] VIO architecture — sliding-window marginalization is understood, but measurement models, bias, initialization, and preintegration are pending
- [ ] SLAM architecture — factor-graph backend and local/global distinction are understood, but a complete sensor-to-map implementation model needs reinforcement
- [ ] Loop closure — optimization role understood; place recognition, verification, and practical failure handling remain pending

## Critical Unresolved Gaps

- [ ] Ground-robot motion models beyond ideal differential drive, including skid-steer parameterization
- [ ] Practical wheel-encoder calibration procedures and parameter identification
- [ ] Quantitative stochastic wheel-slip and odometry noise models
- [ ] Detailed SO(3) exponential/log maps and perturbation conventions
- [ ] Left/right perturbation conventions beyond the established frame-labeled body increment case
- [ ] Formal multi-state ESKF observability and consistency analysis
- [ ] Full IMU propagation model, accelerometer-bias effects, and online bias observability
- [ ] Complete LiDAR geometry and sensor-specific failure modeling
- [ ] Complete scan matching / ICP implementation
- [ ] LiDAR odometry estimator integration
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

Correction: for a rotation matrix, `R^-1 = R^T` because `R` is orthonormal.

### IMU and process model

Initial belief: IMU is simply a process-model sensor.

Correction: IMU produces measurements; estimators commonly use those measurements as inputs to high-rate state propagation.

### Odometry error growth

Initial belief: odometry error grows as `t^2` in general.

Correction: a constant acceleration bias gives a `t^2` position-error term. Different error sources have different growth characteristics.

## Current Focus

```text
LiDAR lever-arm covariance check
→ point-to-line ICP linearization
→ correspondence rejection and robust loss
→ scan-matching quality / degeneracy-aware covariance
→ LiDAR odometry integration with ESKF and factor graph
```
