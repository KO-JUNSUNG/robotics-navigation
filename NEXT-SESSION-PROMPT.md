# Next Learning Session Prompt

> Last updated: 2026-08-25
>
> This file is a session handoff, not the sole source of truth. Reconcile it with `AGENTS.md`, tracking documents, recent topic memos, and the latest Socratic record before teaching.

## How to use

Copy the prompt block below into a new Codex session opened on this repository.

## Prompt

```text
이 저장소의 AGENTS.md를 먼저 읽고 따라라.

이번 세션은 이전 ground-robot navigation / SLAM 학습의 연속이다. 먼저 local-first workflow에 따라 현재 working tree와 branch를 확인하고, 안전하면 최신 master를 pull하라. 그다음 CONTEXT.md, ROADMAP.md, KNOWLEDGE-GAPS.md와 현재 frontier에 직접 관련된 최신 notes/ 및 socratic/ 기록을 읽어 실제 진도를 재구성하라.

현재 신뢰 가능한 학습 범위:

- SE(2) frame, rigid transform, composition, inverse, relative pose
- pose graph / factor graph, MAP와 weighted nonlinear least squares
- Gauss-Newton linearization, sparse Jacobian/Hessian
- variable elimination, fill-in, Schur complement, BA landmark elimination
- marginalization, marginalization prior, sliding-window VIO와 global backend 구분
- differential-drive kinematics와 wheel-encoder odometry
- nonholonomic constraint, slip/calibration failure, first-order covariance propagation
- SO(3) rotation-matrix 의미와 SE(3) frame transformation
- LiDAR extrinsic과 lever-arm effect
- accelerometer specific force, stationary/free-fall 구분
- gravity가 roll/pitch는 제약하지만 yaw는 제약하지 못하는 이유
- gyro bias 누적과 body-frame relative rotation composition
- ESKF nominal/error state, SO(3) correction injection, error-mean reset
- covariance를 reset하지 않는 이유, Kalman-gain과 estimator overconfidence

현재 frontier:

IMU propagation이 yaw error와 gyro-bias error 사이의 cross-covariance를 어떻게 만드는지, 그리고 wheel/LiDAR yaw measurement가 그 correlation을 통해 직접 측정하지 않은 gyro bias를 어떻게 간접 보정하는지 이해하는 단계다.

다음 학습 순서:

1. 작은 개념적 또는 수치적 예제로 yaw-error/gyro-bias cross-covariance를 설명한다.
2. H가 bias를 직접 측정하지 않아도 Kalman gain의 bias row가 non-zero가 될 수 있는 이유를 연결한다.
3. 실제 회전과 bias를 구분하기 위한 excitation 및 observability를 다룬다.
4. 과신된 covariance와 잘못된 wheel/LiDAR measurement가 consistency에 미치는 영향을 다룬다.
5. 이해가 안정되면 concrete IMU + wheel + LiDAR ESKF propagation/update cycle로 확장한다.

다음 항목은 retrieval에서 실제 gap이 드러나지 않는 한 처음부터 반복하지 마라:

- wheel odometry 기본식
- SO(3)/SE(3) 기본 정의와 transform inverse
- accelerometer stationary/free-fall 설명
- 기본적인 ESKF nominal/error-state 및 injection/reset 설명
- Schur complement와 marginalization

Skew matrix나 SO(3) exponential map의 손계산은 현재 learner의 관심사가 아니며, 구현상 필요할 때만 다시 다룬다.

Socratic 방식으로 시작하되 긴 강의부터 하지 마라. 먼저 2~3개의 짧은 retrieval 질문으로 prerequisite가 남아 있는지 확인하고, learner가 reasoning한 다음 필요한 부분만 교정하라.

세션 종료 시 learner가 "오늘 치 공부 끝났다. 다음 세션에서 이어서 할게."라고 말하면, 학습 증거에 따라 notes/, socratic/, CONTEXT.md, ROADMAP.md, KNOWLEDGE-GAPS.md를 필요한 만큼 갱신하고 이 NEXT-SESSION-PROMPT.md도 새로운 frontier와 정확한 다음 시작점으로 반드시 갱신하라. 로컬에서 diff를 검증한 후 commit하고 push하라.
```

## Current handoff boundary

The latest session ended immediately before a concrete explanation of:

```text
gyro-bias error
→ yaw propagation error
→ yaw/bias cross-covariance
→ wheel or LiDAR yaw residual
→ indirect gyro-bias correction
```

Primary references for the next session:

- `notes/eskf-foundations-memo.md`
- `notes/so3-se3-sensor-frames-memo.md`
- `socratic/2026-08-25-ground-robot-kinematics-and-so3.md`
- `KNOWLEDGE-GAPS.md`
- `ROADMAP.md`
