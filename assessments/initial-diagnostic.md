# Initial SLAM Diagnostic — 2026-08-18

This document preserves the learner's original answers and the mentor's assessment. Initial answers are intentionally not rewritten away: they are part of the learning history.

## Background

The learner has a Statistics bachelor's degree, an Artificial Intelligence master's degree, and practical experience autonomously flying a drone with PX4 Offboard mode. Formal navigation and SLAM knowledge is limited.

---

## A1 — Dead Reckoning / Odometry

### Learner Answer

1. 알 수 없다.
2. 바퀴의 크기와 바퀴가 기울어져 있는 각도를 몰라서
3. 출발할 때 위치를 안다면 현재 위치로부터 바퀴가 어느 방향으로 얼마만큼 굴렀는지를 안다면 대략적인 이동 위치는 알 수 있을 것이다.
4. 오차가 시간이 지날수록 t^2만큼 커지기 때문이다.

### Assessment

Good intuition about dead reckoning: initial pose plus measured motion can provide an estimated current pose. However, odometry error does not universally grow as t^2. A constant acceleration bias produces a t^2 position-error term, but wheel slip, encoder quantization, wheel-radius errors, calibration errors, etc. have different behavior.

### Concepts to revisit

- Ground robot kinematics
- Wheel odometry
- Dead reckoning
- Error propagation

---

## A2 — IMU Integration

### Learner Answer

이론적으로는 p(t) = p(0) + v(0)t + 1/2 at^2 이니, 1/2 * 1 * 1**2 이면 0.5m(x) 이다. 1초 후의 속도는 v(1) = v(0) + at = 0 + 1*1 = 1 이다. IMU만 가지고는 초당 제곱으로 불어나는 오차를 보정할 수단이 없어서 오차가 폭주하게 된다. measurement = ground truth + error term(=bias + noise) 인데 이 error term을 두 번 적분함으로써 오차가 t^2으로 증폭된다.

### Assessment

Strong answer. The important distinction is between bias and noise. Under a constant accelerometer-bias model, integrating acceleration twice creates a t^2 position-error term. In real inertial navigation, attitude error and gravity coupling also matter.

### Concepts to revisit

- IMU measurement model
- Bias vs white noise
- Gravity
- Inertial navigation

---

## A3 — Camera-only Localization

### Learner Answer

불가능할 것 같다. 왜냐하면 같은 이미지를 보고 있더라도 특징점은 약간의 오차를 가질 수 있다. 실제로 이게 특징점이 drift하기도 한다. reference로 삼을 수 있는 구조들이 더 필요할 거 같은데, 그게 내 생각에는 일반적으로 gps이고, lidar로 얻을 수 있는 point들도 아마 같이 사용할 수 있지 않을까 생각하고 있다.

### Assessment

Important misconception. Monocular visual odometry/SLAM is possible. The central issue is not simply feature noise; a monocular camera generally cannot determine metric scale from a single image sequence without additional metric information/assumptions. Feature tracking error, degeneracy, scene structure, and observability are also important.

### Concepts to revisit

- Pinhole camera model
- Feature correspondence
- Epipolar geometry
- Monocular scale ambiguity
- Visual odometry

---

## A4 — Kalman Filter / MAP

### Learner Answer

보통 이 경우에 Kalman filter류의 filter을 사용하는 것으로 알고 있다. error state kalman filter가 현업에서 자주 쓰이는 것으로 알고 있고, 이 kalman filter를 MAP라고 해석하는 방법도 있는 것으로 알고 있다.

### Assessment

Good conceptual direction. Filtering is one estimation approach; nonlinear SLAM often uses MAP/nonlinear least-squares formulations. Kalman filtering and MAP are closely related under specific linear-Gaussian assumptions, but should not be treated as universally identical.

### Concepts to revisit

- Bayesian filtering
- MAP estimation
- EKF / ESKF
- Batch optimization

---

## A5 — Sensor Confidence

### Learner Answer

보통 이 경우에 Kalman filter가 가장 잘 작동하는 환경 중 하나로 알고 있고, 나는 여기서 Lidar 센서를 더 믿겠다. 단순히 생각해서, LiDAR의 표준편차가 아주 작으니 이 쪽의 값이 더 신뢰할만 하다고 생각했다. 전자의 분포가 정규분포라고 가정할 때, N(10.0, 1.0^2)에서 10.5는 충분히 등장할 수 있는 값이지만 N(10.5, 0.1^2)에서 10.0은 등장하는 게 불가능한 값이다.

### Assessment

Strong statistical intuition. Smaller covariance means a measurement is more informative under the assumed probabilistic model. 'Impossible' should be read as 'extremely unlikely' for a Gaussian with nonzero variance.

### Concepts to revisit

- Measurement covariance
- Kalman gain
- Information form

---

## A6 — Why SLAM?

### Learner Answer

Localization과 mapping이 동시에 이루어져야하기 때문에, SLAM이라고 따로 부르는 것이다. 내 위치를 기준으로 맵을 만들고, 그 맵을 기준으로 이동해야 되는데 이가 동시에 이루어지지 않는다면 1) 나는 지도가 없어서 이동할 수가 없어 2) 나는 이동할 수가 없어서 지도를 만들 수가 없어 처럼 좀 이상한 상태가 될 것이다.

### Assessment

Good high-level understanding of the coupling. More precise formulation: SLAM jointly estimates unknown robot poses/trajectory and unknown map/landmarks from sensor observations. The coupling is not simply that a robot cannot move without a map; SLAM is an estimation problem with mutually dependent latent variables.

### Concepts to revisit

- Landmark-based SLAM formulation
- Joint estimation
- Observability

---

## A7 — Initial SLAM Pipeline

### Learner Answer

Sensor(LiDAR, IMU, Camera) = raw datas(cloud points, 카메라 상의 특징점들 etc) -> estimator -> pose -> 이전 timestep의 데이터들과 대비 -> Map?

### Assessment

Useful starting mental model but incomplete. A SLAM system has a geometric front-end that turns raw observations into correspondences/relative transformations/measurement constraints, and a back-end that estimates globally consistent states. Mapping and loop closure are also distinct components.

### Concepts to revisit

- SLAM front-end
- SLAM back-end
- Scan matching
- Visual odometry
- Factor graph

---

## A8 — 2D Frame Transformation

### Learner Answer

[10,7]일듯. 90도 돌았으니 y축을 기준으로 보고 있고, 그러면 2만큼 전진했을 때 5+2=7이겠지. 수식적으로는 합성 변환이라는 게 있다는 걸 알고, 그게 이동 변환과 회전변환을 동차좌표계에서 결합해 하나의 행렬로 표현하는 방법이라고 들었음. T=[R,t;0,1] = [R_{3x3}, (t_x,t_y,t_z); 0, 1] = [r_{11}, r_{12}, ...] 뭐 이런 식이라고 듣기는 했는데 정확히 기억은 잘 못함. 이게 뭐 x축, y축, z축 별로 각각 회전행렬 자체도 달랐던 거 같았는데 제대로 기억을 못함.

### Assessment

The numerical answer is correct under the stated convention. The important learning point is that the point must first be rotated from robot frame into world frame, then translated. Homogeneous transforms combine rotation and translation. For 3D, rotation about x/y/z has different elementary rotation matrices.

### Concepts to revisit

- Frame notation
- Rigid transforms
- Homogeneous coordinates
- SO(3)/SE(3)

---

## A9 — Inverse Rotation

### Learner Answer

1) 좌표계가 어디를 기준으로 놓을 것인지 + 해당 좌표계에서 얼마만큼 이동했는지가 필요하기 때문
2) A 좌표의 vector을 B 좌표로 바꾸는 rotation?
3) 맞지 않을까? 1대1로 전환이 되어야 설명이 될 것 같은데. 1대1로 전환이 되지 않고 때에 따라서 바뀐다면 변환이라는 표현이 부적절할 것 같음.

### Assessment

Direction of the inverse-transform answer is right. The reason `R_BA = R_AB^{-1}` is not merely that transformations should be one-to-one. Rotation matrices are orthonormal, so `R^{-1} = R^T`. This is a key property of SO(3).

### Concepts to revisit

- Orthonormal matrices
- SO(3)
- Active/passive interpretation

---

## A10 — Motion and Measurement Models

### Learner Answer

1) f = 로봇 물리 모델, h = 관측 모델, 2) f쪽에 가까운 듯? 왜냐하면 외부 환경을 직접 보는 것이 아니라 로봇 자신의 움직임을 측정해서. 3) h. 4) 시스템 모델이 정확해야 이게 최적화되었을 때 정확한 값으로 수렴한다고 확신할 수 있을테니까?

### Assessment

Good. More precise: `f` describes state transition/propagation, while `h` maps state to expected sensor measurement. IMU produces measurements but is commonly used as the input to high-rate propagation. The distinction depends on estimator formulation.

### Concepts to revisit

- State-space models
- IMU propagation
- Measurement models
- Linearization

---

## A11 — Bayesian Posterior

### Learner Answer

bayesian rule로 계산하면 될 거 같은데 귀찮으니 그냥 넘어가고 싶음. 나는 학부때도 세세한 계산에 능하지는 못했어.

### Assessment

Acceptable for the diagnostic. The learner knows the structural relationship `p(x|z) ∝ p(z|x)p(x)`. Repetitive hand calculation is not a priority; use derivations when they illuminate robotics concepts.

### Concepts to revisit

- Gaussian conjugacy intuition
- MAP
- Information weighting

---

## A12 — Why Estimate + Uncertainty?

### Learner Answer

estimate + uncertainty를 가지고 있어야 사후확률을 계속 업데이트할테니까?

### Assessment

Correct. Extend this to sensor fusion: uncertainty/covariance quantifies how strongly a measurement should influence the posterior. It is also essential for prediction, consistency, observability analysis, and decision-making.

---

## A13 — Robot-frame Motion

### Learner Answer

앞으로 1m라는 명령은 robot frame에서의 이동이고, 이 경우 [1,1,pi/2] 이겠지. 90도 변환해씅니 R(theta) = [cos theta, -sin theta; sin theta, cos theta] 인데 이럼 [0, -1; 1, 0] 이고 [0, -1; 1, 0] [1;0] = [0;1] 이니까.

### Assessment

The rotation calculation is correct, but the final pose was incorrect. Starting at `(0,0,0)`, rotate to `pi/2`, then move 1 m forward in the robot frame. The world displacement is `(0,1)`, so the final pose is `(0,1,pi/2)`, not `(1,1,pi/2)`. The important insight the learner did demonstrate is correct: a robot-frame displacement must be rotated into the world frame before adding it to world position.

### Concepts to revisit

- Pose composition
- Body-frame vs world-frame velocity/displacement
- SE(2)

---

## A14 — Relative Pose

### Learner Answer

T_AB = T_AW · T_WB. B -> world -> A 로 표현

### Assessment

Correct. Since `T_WB` maps B to W and `T_AW` maps W to A, their composition maps B to A. Equivalently:

`T_AB = T_WA^{-1} T_WB`.

This is an important foundation for odometry, scan matching, visual odometry, and pose graphs.

### Concepts to revisit

- Transform composition
- Relative pose
- Pose graphs

---

## A15 / A16 — Next Unresolved Questions

These questions were introduced after the initial diagnostic and are the immediate continuation of the study.

### A15 — Pose and Landmark Relationship

Given a world-frame landmark `^W p` and robot pose `T_WR`, derive the relationship to the landmark expressed in robot coordinates `^R p`. Determine whether knowing pose is sufficient to recover landmark position and vice versa.

### A16 — SLAM as Joint Estimation

When both robot pose and landmark position are unknown but a sensor measures the landmark in the robot frame, formulate the observation as a constraint of the form:

`^R p = f(T_WR, ^W p) + noise`.

Explain why pose and map become mutually coupled unknowns, and why each observation can be interpreted as a constraint in an estimation/optimization problem.

---

## Overall Initial Assessment

The learner has a strong statistical foundation and useful intuition about uncertainty, sensor fusion, and state estimation. The primary weakness is robotics-specific geometry and notation rather than mathematics itself.

Priority order:

1. Coordinate frames and rigid-body transformations
2. SO(3)/SE(3)
3. Ground robot kinematics and odometry
4. Bayesian state estimation → EKF/ESKF
5. Nonlinear optimization
6. Factor graphs / pose graphs
7. LiDAR geometry and scan matching
8. Visual geometry and visual odometry
9. VIO
10. Full SLAM architecture and ground-robot system design
