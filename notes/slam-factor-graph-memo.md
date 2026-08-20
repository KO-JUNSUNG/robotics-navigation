# SLAM / Factor Graph Memo

> 핵심 복습용 압축 memo. 상세 Socratic dialogue는 저장하지 않는다.

## 1. Loop closure

같은 landmark/장소를 다시 관측하면 과거 pose와 현재 pose 사이에 새로운 constraint가 생긴다.

Landmark 관측:

`T_WR1 ^R1 p_L ≈ T_WR5 ^R5 p_L`

Residual:

`r_L = T_WR1 ^R1 p_L - T_WR5 ^R5 p_L`

관측을 scan matching 등으로 상대 pose constraint로 표현하면:

`T_R1R5^loop ≈ T_WR1^{-1} T_WR5`

Loop closure는 odometry drift를 전체 trajectory가 constraint를 함께 만족하도록 보정하게 만든다.

**주의:** loop closure가 world frame 자체를 고정하는 것은 아니다.

## 2. Pose graph

Odometry는 pose-pose constraint:

`z_12 ≈ T_WR1^{-1} T_WR2`

Measurement model:

`h(T_WR1,T_WR2) = T_WR1^{-1}T_WR2`

Residual의 개념:

`r_12 = z_12 - h(X)`

실제 SE(2)/SE(3)에서는 transformation matrix를 단순히 빼기보다 Lie group의 local vector representation을 사용한다.

## 3. Factor graph

각 measurement는 관련된 state variable만 직접 연결한다.

`z_12 → (x_1,x_2)`

`z_3L → (x_3,l_L)`

같은 landmark를 여러 pose에서 관측하면 여러 pose-landmark factors가 생겨 graph를 연결하고 state를 공동 추정할 수 있다.

Loop closure는 시간적으로 떨어진 pose 사이의 장거리 constraint를 추가한다.

## 4. Bayesian → optimization

SLAM의 목표는 posterior를 추정하는 것:

`p(X,L | Z) ∝ p(Z | X,L) p(X,L)`

Measurement들이 조건부 독립이면 likelihood는 여러 local factor로 factorization된다.

Gaussian measurement:

`z_i ~ N(h_i(X), Σ_i)`

그러면 MAP/MLE는 weighted nonlinear least squares 형태로 연결된다:

`X* = argmin_X Σ_i r_i^T Σ_i^{-1} r_i`

- `Σ_i`: uncertainty / covariance
- `Σ_i^{-1}`: information matrix / weight
- 신뢰도가 높은 measurement(작은 covariance)는 더 강하게 반영된다.

핵심 연결:

`Gaussian → likelihood → MAP/MLE → weighted nonlinear least squares → factor graph optimization`

## 5. DOF / observability / gauge freedom

Measurement 하나가 state의 모든 자유도를 결정하지는 않는다.

`H = ∂h/∂X`

Jacobian의 rank는 locally independent constraint가 state를 얼마나 제약하는지와 관련된다.

SLAM에서는 전체 world frame을 통째로 같은 rigid transformation `G`로 바꿔도 relative constraints가 변하지 않는다:

`T'_Wi = G T_Wi`

따라서 gauge freedom이 존재한다.

보통 첫 pose를 고정:

`T_WR1 = I`

이것이 gauge fixing이다.

**Loop closure는 drift를 줄이지만 gauge freedom을 자동으로 제거하지 않는다.**

## 6. 핵심 mental model

`Sensor → measurement z → measurement model h(X) → residual r → uncertainty Σ → weighted cost → optimization`

SLAM은 로봇 pose와 landmark라는 unknown들을 여러 sensor constraint로 연결하고, 전체 constraint를 가장 잘 만족하는 상태를 추정하는 nonlinear estimation problem이다.
