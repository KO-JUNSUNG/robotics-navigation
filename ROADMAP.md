# 2026-08-19 ~ 2026-09-01 Study Roadmap

Goal: build the theoretical and practical foundation required to design and apply SLAM to a ground robot.

> This roadmap is sequencing guidance, not proof of the current learning frontier. Status is reconciled from the most recent topic-specific notes and Socratic records.

| Original Date | Day | Topic | Evidence-based Status |
|---|---:|---|---|
| 2026-08-18 | 0 | Initial diagnostic, SLAM overview | Completed |
| 2026-08-19 | 1 | Coordinate frames, rigid-body transformations | Completed for 2D / SE(2) |
| 2026-08-20 | 2 | SO(3), SE(3), rotation mathematics | Introductory SO(3) / SE(3) frame-transformation foundation completed |
| 2026-08-21 | 3 | Ground robot kinematics, odometry, motion models | Completed for differential drive and wheel odometry foundation |
| 2026-08-22 | 4 | Bayesian state estimation, MAP/MLE, Gaussian uncertainty | Conceptual foundation completed |
| 2026-08-23 | 5 | Kalman Filter, EKF, ESKF | Cross-covariance, indirect bias correction, observability/consistency foundation completed; full Jacobians and implementation pending |
| 2026-08-24 | 6 | Nonlinear least squares, Gauss-Newton, LM | Nonlinear least squares, linearization, and Gauss-Newton structure completed conceptually; LM pending |
| 2026-08-25 | 7 | Factor graphs, pose graphs, constraints | Completed conceptually, including sparsity, elimination, and Schur complement |
| 2026-08-26 | 8 | LiDAR geometry, scan matching, ICP, LiDAR odometry | Conceptual scan geometry, ICP behavior, degeneracy, deskew, and extrinsic conversion in progress |
| 2026-08-27 | 9 | Camera model, features, epipolar geometry | Pending |
| 2026-08-28 | 10 | Visual odometry, triangulation, scale ambiguity | Scale-ambiguity intuition established; remaining topics pending |
| 2026-08-29 | 11 | IMU + camera, VIO, bias, initialization, preintegration | Sliding-window marginalization understood conceptually; sensor model, initialization, and preintegration pending |
| 2026-08-30 | 12 | SLAM architecture, front-end, back-end, loop closure | Factor-graph and local-estimator/global-backend distinction established; full practical architecture pending |
| 2026-08-31 | 13 | Ground robot SLAM systems, LiDAR/Visual/VIO/LIO comparison | Pending |
| 2026-09-01 | 14 | Capstone: design the target ground-robot SLAM system | Pending |

## Current Learning Frontier

The next active prerequisite chain is:

```text
completed differential-drive, SO(3), and introductory SE(3) foundations
→ completed ESKF cross-covariance, indirect-state correction, observability/consistency foundation
→ completed conceptual IMU / wheel / LiDAR propagation-update cycle
→ LiDAR point-to-line ICP linearization and robust correspondence handling
→ LiDAR odometry measurement integration
```

The learner already has a working conceptual understanding of:

- Coordinate-frame direction and SE(2) transform composition.
- Odometry as a relative-pose constraint.
- Pose graphs and factor graphs.
- Gaussian measurement factors, covariance/information, and MAP as weighted nonlinear least squares.
- Linearization, Jacobian sensitivity, Gauss-Newton Hessian sparsity.
- Variable elimination, fill-in, Schur complement, and landmark elimination in Bundle Adjustment.
- Marginalization, marginalization priors, sliding-window VIO, and the local-estimator/global-pose-graph distinction.
- Differential-drive kinematics, turning radius, and wheel-encoder odometry integration.
- Nonholonomic constraints, wheel slip, calibration errors, multi-sensor disagreement, and first-order covariance propagation.
- SO(3) / SE(3) frame transformations, inverse transforms, and sensor extrinsic composition.
- LiDAR lever-arm effects and accelerometer specific-force / gravity reasoning.
- ESKF nominal/error state, bias-state motivation, correction injection/reset, and Kalman-gain intuition.

These topics should be connected briefly when useful rather than automatically retaught. Advanced backend topics such as Bayes Tree, iSAM2, FEJ, and detailed elimination ordering remain deferred until a practical need or prerequisite chain justifies them.

## Near-term Sequence

1. Complete point-to-line ICP linearization, robust correspondence handling, and scan-matching quality evaluation.
2. Connect LiDAR odometry measurements and covariance to the ESKF and factor-graph viewpoints.
3. Return to full ESKF propagation/update Jacobians when implementation requires them.
4. Build camera and visual-odometry prerequisites.
5. Study IMU fusion, VIO / LIO, and practical SLAM architecture.
6. Perform sensor/stack selection, experiment design, and capstone system design.

This sequence is prerequisite-driven. It is not necessary to follow the original calendar dates literally.

## Daily Study Pattern

Target: approximately 2–3 hours/day.

1. Concept introduction: ~40 min
2. Socratic questioning: ~40 min
3. Targeted mathematical derivation: ~30 min
4. Implementation/system connection: ~30–60 min

## Milestone

The learner should be able to explain and reason through:

- What each sensor measures.
- Coordinate-frame transformations and relative pose.
- Robot motion and odometry models.
- Bayesian state estimation and uncertainty.
- KF/EKF/ESKF at a conceptual and implementation level.
- MAP as nonlinear optimization.
- Factor graphs and pose graphs.
- LiDAR scan matching and ICP.
- Visual geometry and visual odometry.
- Why IMU + camera forms a useful VIO system.
- SLAM front-end/back-end/loop closure.
- How to choose and architect a SLAM stack for a ground robot.

## Important Note

The goal is not to master every derivation immediately or treat the original sprint dates as deadlines. The goal is to build a coherent mental model and enough mathematical and systems vocabulary to study, implement, test, and debug real ground-robot SLAM without treating it as a black box.
