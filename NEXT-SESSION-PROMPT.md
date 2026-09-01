# Next Learning Session Prompt

> Last updated: 2026-09-01
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

추가로 신뢰 가능해진 범위:

- IMU propagation의 yaw-error / gyro-bias cross-covariance 생성
- yaw-only wheel/LiDAR measurement의 indirect gyro-bias correction
- cross-covariance와 observability의 구분
- additive bias / multiplicative scale error와 excitation
- measurement consistency, adaptive covariance, rejection, duplicate update failure
- conceptual asynchronous IMU / wheel / LiDAR ESKF cycle
- LiDAR scan frame, ICP correspondence, point-to-line normal, corridor degeneracy
- repeated-structure local minima, scan deskew, LiDAR-to-robot extrinsic motion conversion

현재 frontier:

LiDAR lever arm이 yaw uncertainty를 position uncertainty로 바꾸는 관계를 확인한 뒤, point-to-line ICP의 작은-pose linearization과 robust correspondence handling으로 진행하는 단계다.

다음 학습 순서:

1. 직전 질문 \(r=0.2\,\mathrm m\), \(\sigma_\theta=0.1\,\mathrm{rad}\)에서 \(\sigma_p\approx r\sigma_\theta\)를 짧게 retrieval한다.
2. point-to-line ICP residual을 작은 2D pose increment로 linearize하며 translation/yaw Jacobian의 물리적 의미를 설명한다.
3. nearest-neighbor rejection, maximum correspondence distance, trimming, robust loss를 다룬다.
4. convergence, overlap, residual, Hessian eigenstructure로 scan-matching quality와 degeneracy-aware covariance를 연결한다.
5. LiDAR relative-pose measurement를 body frame과 timestamp에 맞춰 ESKF/factor graph에 넣는 방법으로 확장한다.

다음 항목은 retrieval에서 실제 gap이 드러나지 않는 한 처음부터 반복하지 마라:

- wheel odometry 기본식
- SO(3)/SE(3) 기본 정의와 transform inverse
- accelerometer stationary/free-fall 설명
- 기본적인 ESKF nominal/error-state 및 injection/reset 설명
- Schur complement와 marginalization
- ESKF cross-covariance 수치 예제와 correlation/observability 구분
- ICP frame 기본식, corridor normal intuition, deskew 필요성

Skew matrix나 SO(3) exponential map의 손계산은 현재 learner의 관심사가 아니며, 구현상 필요할 때만 다시 다룬다.

Socratic 방식으로 시작하되 긴 강의부터 하지 마라. 먼저 2~3개의 짧은 retrieval 질문으로 prerequisite가 남아 있는지 확인하고, learner가 reasoning한 다음 필요한 부분만 교정하라.

세션 종료 시 learner가 "오늘 치 공부 끝났다. 다음 세션에서 이어서 할게."라고 말하면, 학습 증거에 따라 notes/, socratic/, CONTEXT.md, ROADMAP.md, KNOWLEDGE-GAPS.md를 필요한 만큼 갱신하고 이 NEXT-SESSION-PROMPT.md도 새로운 frontier와 정확한 다음 시작점으로 반드시 갱신하라. 로컬에서 diff를 검증한 후 commit하고 push하라.
```

## Current handoff boundary

The latest session ended at the concrete relation:

```text
LiDAR lever arm r
→ yaw uncertainty sigma_theta
→ approximate position uncertainty sigma_p ≈ r sigma_theta
→ covariance must be transformed with the extrinsic, not copied
```

Primary references for the next session:

- `notes/eskf-foundations-memo.md`
- `notes/lidar-scan-matching-foundations-memo.md`
- `notes/so3-se3-sensor-frames-memo.md`
- `socratic/2026-08-26-eskf-cross-covariance-and-lidar-scan-matching.md`
- `KNOWLEDGE-GAPS.md`
- `ROADMAP.md`
