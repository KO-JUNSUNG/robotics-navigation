# Robotics Navigation & SLAM Study

## Objective

지상 로봇에 SLAM을 적용하는 임무를 수행하기 위해 항법, 상태추정, 센서 융합, LiDAR/Visual/VIO/SLAM의 이론적 기반과 실제 시스템 설계 능력을 확보한다.

## Learner Background

- Statistics B.S.
- Artificial Intelligence M.S.
- PX4 Offboard mode를 사용한 드론 자율비행 경험
- Robot navigation/SLAM은 초보 단계
- 확률/통계 및 AI 배경은 비교적 강함
- 선호 학습법: Socratic questioning

## Current Goal

2026-09-01까지 SLAM을 단순히 라이브러리 사용법으로 아는 것이 아니라, 다음을 설명하고 설계할 수 있는 수준에 도달한다.

- 센서가 무엇을 측정하는가
- pose와 map을 어떤 latent variable로 볼 수 있는가
- motion/measurement model은 무엇인가
- estimation과 nonlinear optimization은 어떻게 연결되는가
- LiDAR SLAM, Visual SLAM, VIO/LIO의 구조와 차이
- ground robot에 적합한 SLAM architecture를 설계하는 방법

## Learning Principles

1. 개념 → 직관 → 수식 → 구현 순서로 학습한다.
2. 이해하지 못한 수식을 암기하지 않는다.
3. 모든 transformation에는 source/destination frame을 명시한다.
4. 오개념과 초기 답변을 삭제하지 않고 학습 이력으로 보존한다.
5. Socratic questioning으로 지식의 빈틈을 발견하고 보완한다.
6. 통계/확률의 기초를 불필요하게 반복하지 않고 robotics와 연결하는 데 집중한다.
7. 특정 SLAM library 사용법보다 underlying estimation/geometry를 먼저 이해한다.
8. 실제 로봇 시스템에서 각 이론이 어느 module에 해당하는지 계속 연결한다.

## Current Status

2026-08-18 initial diagnostic 진행 중. Coordinate frames와 rigid-body transformation을 처음 본격적으로 학습하기 시작했다.

현재 핵심 gap:
- coordinate frame conventions
- rigid-body transformation
- SO(3)/SE(3)
- robot kinematics/odometry
- EKF/ESKF의 상세 구조
- nonlinear least squares
- factor graph / pose graph
- LiDAR geometry / scan matching / ICP
- visual geometry / epipolar geometry
- VIO
- SLAM front-end/back-end/loop closure

현재 강점:
- probability/statistics intuition
- Bayesian inference intuition
- sensor uncertainty/covariance intuition
- Kalman filter의 기본 개념
- machine learning background
- PX4/Offboard 실제 자율비행 경험

## How to Continue in a New Chat

1. Read this README.
2. Read `CONTEXT.md`.
3. Read `ROADMAP.md` and `KNOWLEDGE-GAPS.md`.
4. Read the latest file under `socratic/`.
5. Continue from the first unresolved Socratic question rather than restarting the curriculum.

## Repository Structure

```text
README.md
CONTEXT.md
ROADMAP.md
KNOWLEDGE-GAPS.md
assessments/
socratic/
notes/
experiments/
references/
```
