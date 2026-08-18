# 2026-08-19 ~ 2026-09-01 Study Roadmap

Goal: build the theoretical and practical foundation required to design and apply SLAM to a ground robot.

| Date | Day | Topic | Status |
|---|---:|---|---|
| 2026-08-18 | 0 | Initial diagnostic, SLAM overview | In progress |
| 2026-08-19 | 1 | Coordinate frames, rigid-body transformations | Pending |
| 2026-08-20 | 2 | SO(3), SE(3), rotation mathematics | Pending |
| 2026-08-21 | 3 | Ground robot kinematics, odometry, motion models | Pending |
| 2026-08-22 | 4 | Bayesian state estimation, MAP/MLE, Gaussian uncertainty | Pending |
| 2026-08-23 | 5 | Kalman Filter, EKF, ESKF | Pending |
| 2026-08-24 | 6 | Nonlinear least squares, Gauss-Newton, LM | Pending |
| 2026-08-25 | 7 | Factor graphs, pose graphs, constraints | Pending |
| 2026-08-26 | 8 | LiDAR geometry, scan matching, ICP, LiDAR odometry | Pending |
| 2026-08-27 | 9 | Camera model, features, epipolar geometry | Pending |
| 2026-08-28 | 10 | Visual odometry, triangulation, scale ambiguity | Pending |
| 2026-08-29 | 11 | IMU + camera, VIO, bias, initialization, preintegration | Pending |
| 2026-08-30 | 12 | SLAM architecture, front-end, back-end, loop closure | Pending |
| 2026-08-31 | 13 | Ground robot SLAM systems, LiDAR/Visual/VIO/LIO comparison | Pending |
| 2026-09-01 | 14 | Capstone: design the target ground-robot SLAM system | Pending |

## Daily Study Pattern

Target: approximately 2–3 hours/day.

1. Concept introduction: ~40 min
2. Socratic questioning: ~40 min
3. Targeted mathematical derivation: ~30 min
4. Implementation/system connection: ~30–60 min

## Milestone for 2026-09-01

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

The goal of this sprint is not to master every derivation or implement a production SLAM system by 2026-09-01. The goal is to build a coherent mental model and enough mathematical vocabulary to study and implement real systems without treating them as black boxes.
