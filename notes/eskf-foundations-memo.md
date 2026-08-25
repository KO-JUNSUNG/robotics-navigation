# ESKF Foundations Memo

> 목적: IMU propagation이 왜 bias와 drift를 만들고, ESKF가 nominal state와 작은 error state를 분리하여 measurement correction을 수행하는지를 복원하기 위한 입문 memo. 전체 Jacobian 유도와 구현은 아직 다음 학습 범위다.

## Storyline

\`\`\`text
IMU는 high rate지만 bias와 noise가 누적
→ bias를 고정 상수가 아니라 state로 추정해야 함
→ orientation은 일반 vector처럼 더할 수 없음
→ nominal state + small error state
→ IMU로 nominal state와 covariance propagation
→ wheel / LiDAR residual로 error state update
→ correction을 nominal state에 inject
→ error mean reset, covariance는 보존
→ Kalman gain이 prior와 measurement 신뢰도를 조절
→ cross-covariance를 통해 직접 측정하지 않은 bias도 보정 가능
\`\`\`

## 1. Why bias belongs in the state

Gyro measurement:

\[
\omega_m=\omega_{\text{true}}+b_g+n_g
\]

- \(\omega_m\): measured angular velocity
- \(\omega_{\text{true}}\): true angular velocity
- \(b_g\): gyro bias
- \(n_g\): measurement noise

실제 bias는 temperature, time, vibration, sensor nonlinearity 등에 따라 변할 수 있다. Ground calibration에서 얻은 한 값이 motor vibration이나 thermal equilibrium이 달라진 운용 상태에서 그대로 유지된다고 가정할 수 없다.

단순 bias process model:

\[
b_{g,k+1}=b_{g,k}+w_{bg}
\]

여기서 \(w_{bg}\)는 bias random walk를 나타낸다.

Bias를 state에 넣는 것만으로 자동 추정되는 것은 아니다. True angular velocity와 bias를 구분할 external constraint와 충분한 excitation이 필요하다.

## 2. Gyro-only ambiguity

Gyro가 일정한 값을 출력할 때 gyro alone으로는 그것이:

- constant true rotation
- constant bias
- 두 값의 조합

중 무엇인지 구분할 수 없다.

Wheel odometry는:

\[
\omega_{\text{wheel}}
=
\frac{v_R-v_L}{b}
\]

를 제공할 수 있고, LiDAR scan matching은 static environment가 암시하는 relative rotation을 제공할 수 있다. 이 정보들도 slip, calibration, registration failure가 있으므로 covariance 및 outlier handling이 필요하다.

## 3. Nominal state and error state

대표적인 nominal state:

\[
x=
\left(
p,\ v,\ R,\ b_a,\ b_g
\right)
\]

- \(p\): position
- \(v\): velocity
- \(R\): orientation
- \(b_a\): accelerometer bias
- \(b_g\): gyro bias

작은 error state:

\[
\delta x=
\left(
\delta p,\delta v,\delta\theta,
\delta b_a,\delta b_g
\right)
\]

ESKF의 큰 흐름:

\`\`\`text
IMU
→ nominal state high-rate propagation
→ error covariance propagation
→ wheel / LiDAR / vision measurement
→ residual
→ error-state correction
→ nominal state에 injection
→ error-state mean reset
\`\`\`

## 4. Why orientation error uses three local coordinates

Rotation matrix는:

\[
R^\top R=I,\qquad\det R=1
\]

을 만족해야 한다.

일반 matrix addition:

\[
R_{\text{new}}=R+\delta R
\]

은 orthonormality와 determinant를 보장하지 않는다.

ESKF는 작은 orientation error:

\[
\delta\theta\in\mathbb R^3
\]

를 추정하고, 이를 valid rotation으로 바꾸어 합성한다.

\[
R_{\text{new}}
=
R\operatorname{Exp}([\delta\theta]_\times)
\]

\(R\in SO(3)\)이고 \(\operatorname{Exp}([\delta\theta]_\times)\in SO(3)\)이므로 그 곱도 \(SO(3)\)에 남는다.

Skew matrix나 exponential map의 수동 계산은 현재 필수 범위가 아니다. 실제 implementation에서 필요할 때 상세히 다룬다.

## 5. Bias residual intuition

IMU yaw increment를 단순화하면:

\[
\Delta\theta_{\text{IMU}}
=
(\omega_m-\hat b_g)\Delta t
\]

Wheel yaw increment와의 residual을:

\[
r_\theta
=
\Delta\theta_{\text{wheel}}
-
\Delta\theta_{\text{IMU}}
\]

라고 할 수 있다.

만약 \(\hat b_g<b_g\)라면 corrected IMU rate가 너무 크고:

\[
r_\theta<0
\]

가 된다. IMU rotation prediction을 줄이려면 \(\hat b_g\)를 증가시키는 방향의 correction이 필요하다.

## 6. Injection and reset

Measurement update가 \(\delta\hat x\)를 추정하면 nominal state에 주입한다.

\[
p\leftarrow p+\delta\hat p
\]

\[
b_g\leftarrow b_g+\delta\hat b_g
\]

\[
R\leftarrow
R\operatorname{Exp}([\delta\hat\theta]_\times)
\]

그 뒤:

\[
\delta\hat x\leftarrow0
\]

으로 error-state mean을 reset한다.

이유는 correction이 이미 nominal state에 포함되었기 때문이다. Error mean을 남겨두면 다음 update에서 같은 correction을 다시 적용하는 double counting이 생긴다.

## 7. Resetting the mean does not remove uncertainty

Error-state mean reset:

\[
\delta\hat x\leftarrow0
\]

은 state가 완벽히 정확하다는 뜻이 아니다. Covariance \(P\)는 correction 이후에도 남는 uncertainty를 표현한다.

따라서:

\[
P\neq0
\]

이며, 엄밀한 ESKF에서는 새로운 error coordinate에 맞게 reset Jacobian으로 covariance를 변환한다.

Error mean과 covariance를 동시에 0으로 만드는 것은:

> 현재 nominal state를 절대적으로 확신한다.

라는 잘못된 선언이다.

## 8. Kalman gain and trust

Kalman gain:

\[
K
=
PH^\top
\left(
HPH^\top+R
\right)^{-1}
\]

- \(P\): predicted state covariance
- \(H\): measurement sensitivity Jacobian
- \(R\): measurement noise covariance
- \(K\): residual을 correction에 반영하는 gain

Correction:

\[
\delta x=Kr
\]

큰 residual이 \(H\)를 크게 만드는 것은 아니다.

- \(r=z-h(x)\): measurement와 prediction의 차이
- \(H=\partial h/\partial x\): state change에 대한 predicted measurement sensitivity

1D에서 \(H=1\)이면:

\[
K=\frac{P}{P+R}
\]

이다.

\(P\)를 비현실적으로 작게 유지하면 \(K\)가 작아져 external measurement를 무시한다. 실제 error는 큰데 covariance만 작은 estimator는 overconfident하고 inconsistent하다.

## 9. Covariance propagation and update

Propagation:

\[
P_{k+1}^{-}
=
F_kP_k^{+}F_k^\top+Q_k
\]

- \(F_k\): 기존 error를 다음 시점으로 전파
- \(Q_k\): IMU noise와 bias process noise 등 새 uncertainty

Measurement update의 단순형:

\[
P^+
\approx
(I-KH)P^-
\]

IMU-only propagation에서 uncertainty가 커지는 이유를 모든 error가 \(t^2\)로 자란다고 설명하면 안 된다.

- constant accelerometer bias는 position에 \(t^2\) 항을 만들 수 있다.
- gyro bias는 orientation error를 누적시킨다.
- orientation error는 gravity compensation과 position에 coupling된다.
- white noise와 bias random walk는 서로 다른 성장 형태를 가진다.
- covariance는 \(F\)를 통한 coupling과 \(Q\) 추가로 전파된다.

## 10. Direct observation and indirect correction

Measurement가 직접 관측하는 state 방향의 uncertainty가 주로 감소한다. 하지만 covariance의 off-diagonal cross-correlation을 통해 직접 측정되지 않은 state도 간접 correction될 수 있다.

예:

\`\`\`text
gyro-bias error
→ IMU propagation 중 yaw error에 영향
→ yaw error와 gyro-bias error 사이 cross-covariance 생성
→ wheel / LiDAR yaw residual
→ yaw correction
→ correlated gyro bias도 correction
\`\`\`

Bias를 measurement equation에 직접 넣지 않아도 correlation이 있으면 Kalman gain의 bias row가 non-zero가 될 수 있다.

이 mechanism과 observability 조건은 다음 학습 frontier다.

## 11. Consistency and failure handling

큰 Kalman gain이 항상 좋은 것은 아니다. Scan matching이 틀렸다면 큰 correction이 estimator를 망칠 수 있다.

실제 estimator에는 다음이 중요하다.

- realistic \(P,Q,R\)
- innovation / residual consistency check
- gating and outlier rejection
- robust handling of wheel slip and registration failure
- time synchronization and correct extrinsics
- observability-aware state design

## Current boundary

현재 신뢰 가능한 범위:

- nominal/error-state motivation
- bias as a changing state
- valid SO(3) error injection
- injection/reset and mean/covariance distinction
- Kalman-gain trust intuition
- direct versus correlation-mediated correction concept

아직 미완료:

- complete continuous/discrete IMU error dynamics
- \(F,G,Q\) derivation
- accelerometer-bias coupling details
- reset Jacobian
- measurement-specific \(H\)
- cross-covariance numerical example
- observability and consistency analysis
- implementation in a ROS 2 / ground-robot stack

## Final mental model

> ESKF는 IMU로 nominal state를 빠르게 propagation하면서 작은 Euclidean error state와 covariance를 추적한다. External measurement가 residual을 제공하면 error correction을 nominal state에 주입하고 error mean을 reset하되 uncertainty는 보존한다. Bias처럼 직접 측정되지 않는 state는 propagation 중 만들어진 cross-covariance를 통해 간접적으로 보정될 수 있으며, 이 과정의 observability와 consistency가 다음 핵심 과제다.
