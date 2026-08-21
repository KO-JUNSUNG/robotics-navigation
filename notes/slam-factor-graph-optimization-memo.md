# SLAM Factor Graph — Optimization / Sparsity Memo

> 목적: 기존 `slam-factor-graph-memo.md`의 다음 단계로서, **Factor Graph가 nonlinear optimization과 sparse computation으로 어떻게 연결되고, 왜 Schur Complement와 marginalization이 필요한지** 나중에 storyline으로 복원하기 위한 memo.

## Storyline

```text
Factor Graph
→ 각 factor는 일부 variable만 본다
→ residual을 state에 대해 미분하면 Jacobian
→ 연결되지 않은 variable에 대한 derivative는 0
→ Sparse Jacobian
→ Gauss-Newton에서 H ≈ JᵀΩJ
→ Sparse Hessian / Information Matrix
→ Sparse optimization으로 대규모 SLAM 계산 가능
→ Variable을 제거하면 주변 variable 사이에 새로운 coupling 발생
→ Fill-in
→ 제거할 variable의 정보는 버리지 않고 남은 variable에 흡수해야 함
→ Schur Complement
→ Bundle Adjustment에서는 landmark를 효율적으로 제거 가능
→ 시간에 따라 state가 계속 쌓이는 VIO에서는 오래된 state를 제거해야 함
→ 단순 삭제하면 과거 정보가 사라짐
→ Marginalization
→ 과거 정보는 prior로 남김
→ Sliding-window estimation
→ bounded computation ↔ 과거 nonlinear factor를 다시 재선형화할 수 없는 trade-off
```

---

## 1. Nonlinear Least Squares → Linearization

SLAM의 전체 cost는

`J_cost(X) = Σ_i r_i(X)^T Ω_i r_i(X)`

이다.

Residual이 nonlinear이면 현재 estimate `X_k` 주변에서 작은 update `ΔX`에 대해

`r(X_k + ΔX) ≈ r(X_k) + J ΔX`

로 local linearization한다.

핵심은 nonlinear 함수의 미분이 단순히 복잡해서가 아니다.

> **Nonlinear optimization을 현재 추정값 주변의 local linear problem으로 바꾸어 반복적으로 풀기 위해 linearization한다.**

```text
X_k
→ linearize
→ J 계산
→ linear problem 해결
→ ΔX 계산
→ X_{k+1} = X_k + ΔX
→ 다시 linearize
→ 반복
```

Gauss-Newton 자체의 알고리즘보다는, 이 방법이 SLAM의 factor 구조에 어떻게 연결되는지가 중요하다.

---

## 2. Jacobian = Residual Sensitivity

Jacobian은 단순한 "편미분 행렬"이 아니라

> **state를 조금 움직였을 때 residual이 얼마나, 어느 방향으로 변하는지를 나타내는 local sensitivity map**

으로 이해한다.

1D odometry 예:

`r = 5 - (x2 - x1) = 5 - x2 + x1`

따라서

`∂r/∂x1 = +1`

`∂r/∂x2 = -1`

이다.

즉 `x1`을 +1 움직이면 residual은 +1 변하고, `x2`를 +1 움직이면 residual은 -1 변한다.

일반적으로

`Δr ≈ J ΔX`

이다.

---

## 3. Factor Graph → Sparse Jacobian

예를 들어

```text
x1 ── x2 ── x3
```

이고

`r12 = r12(x1,x2)`

`r23 = r23(x2,x3)`

라면 전체 Jacobian은 구조적으로

```text
J = [ *  *  0
      0  *  * ]
```

가 된다.

왜냐하면

`∂r12/∂x3 = 0`

`∂r23/∂x1 = 0`

이기 때문이다.

즉 **factor와 관계없는 variable에 대한 residual의 편미분은 0**이다.

따라서

```text
Factor Graph의 local connectivity
→ residual dependency
→ Jacobian의 non-zero structure
→ Sparse Jacobian
```

이 된다.

Jacobian의 sparsity는 우연한 수치적 특성이 아니라 **Factor Graph의 연결 구조가 그대로 수학에 반영된 결과**다.

---

## 4. Jacobian → Hessian / Information Matrix

Gauss-Newton에서는 linearized weighted least squares로부터

`H ΔX = -b`

형태의 선형 시스템을 얻는다.

개념적으로

`H ≈ Jᵀ Ω J`

이다.

- `J`: residual sensitivity
- `Ω = Σ⁻¹`: measurement information / confidence
- `H`: Gauss-Newton Hessian approximation이며 local information structure
- `b`: 현재 residual에서 만들어지는 gradient 계열 항
- `ΔX`: state correction

예를 들어 `x1 -- x2 -- x3`라면 `H`도 구조적으로

```text
H ≈ [ *  *  0
      *  *  *
      0  *  * ]
```

와 같은 sparse structure를 가진다.

따라서

```text
Factor Graph connectivity
→ Jacobian sparsity
→ Hessian / information matrix sparsity
```

가 연결된다.

실제 SLAM에서는 state가 수천~수만 개여도 하나의 measurement는 보통 소수의 variable만 연결한다. 따라서 0인 block을 저장하고 계산할 필요가 없는 sparse linear algebra를 사용하여 대규모 optimization을 효율적으로 풀 수 있다.

---

## 5. Variable Elimination → Fill-in

다음 구조를 생각한다.

```text
x1 ── x2 ── x3
```

`x2`는 `f12(x1,x2)`와 `f23(x2,x3)`에 동시에 참여한다.

`x2`를 제거한다고 해서 `x2`가 제공하던 정보를 버리는 것은 아니다. `x2`의 효과를 남은 variable 사이의 새로운 effective constraint로 표현한다.

```text
제거 전:
x1 ── x2 ── x3

제거 후:
x1 ───────── x3
```

이처럼 elimination 과정에서 원래 없던 non-zero coupling이 생기는 것을 **fill-in**이라고 한다.

중요:

> fill-in은 pose들이 시간 순서로 연속되기 때문에 생기는 것이 아니라, **제거되는 variable이 여러 factor를 통해 주변 variable들을 공통으로 연결하고 있었기 때문**이다.

Landmark에서도 동일하다.

```text
        L
      / | \
     x1 x2 x3
```

`L`을 제거하면 `x1-x2`, `x1-x3`, `x2-x3` 사이에 effective coupling이 생길 수 있다. 즉 variable 수는 줄지만 reduced system은 더 dense해질 수 있다.

---

## 6. Schur Complement — Variable은 제거하되 정보는 남긴다

Gauss-Newton linear system을 유지할 variable `x`와 제거할 variable `y`로 나눈다.

```text
[ H_xx  H_xy ] [ Δx ]   [ -b_x ]
[ H_yx  H_yy ] [ Δy ] = [ -b_y ]
```

두 번째 block equation은

`H_yx Δx + H_yy Δy = -b_y`

이다.

이를 `Δy`에 대해 풀면

`Δy = -H_yy⁻¹ b_y - H_yy⁻¹ H_yx Δx`

이다.

이 식을 첫 번째 block equation에 대입하면

`(H_xx - H_xy H_yy⁻¹ H_yx) Δx = -(b_x - H_xy H_yy⁻¹ b_y)`

를 얻는다.

따라서 reduced Hessian은

`H_reduced = H_xx - H_xy H_yy⁻¹ H_yx`

이다.

이것이 Schur Complement다.

### 물리적 의미

`y`를 그냥 삭제한 것이 아니다.

> **y 자체는 optimization variable에서 제거하지만, y가 x에 제공하던 constraint/information의 효과는 reduced system에 흡수한다.**

즉

```text
Variable 제거
≠ Information 삭제
```

이다.

---

## 7. 왜 Bundle Adjustment에서는 Landmark를 먼저 제거하는가?

전체 state를 pose와 landmark로 나누면

```text
H = [ H_PP  H_PL
      H_LP  H_LL ]
```

가 된다.

전형적인 Bundle Adjustment에서는 하나의 visual measurement가 하나의 pose와 하나의 landmark를 연결하지만, landmark-landmark를 직접 연결하는 factor는 보통 없다.

따라서 `H_LL`은 landmark별 **block diagonal** 구조를 가진다.

```text
H_LL = [ H_L1L1     0        0
            0     H_L2L2     0
            0        0     H_L3L3 ]
```

3D landmark라면 각각 작은 `3×3` block이 될 수 있다.

따라서 `H_LL⁻¹`은 거대한 dense inverse를 계산하는 것이 아니라 각 작은 landmark block을 독립적으로 처리할 수 있다.

그래서

```text
Pose + 매우 많은 Landmark
→ H_LL이 block diagonal
→ Landmark를 Schur Complement로 효율적으로 eliminate
→ Pose-only reduced system
→ Pose update 계산
→ 필요하면 landmark update back-substitution
```

이라는 전략이 가능하다.

중요한 표현:

> **Landmark라서 본질적으로 항상 독립인 것이 아니라, 전형적인 BA factor graph에서 landmark-landmark factor가 없기 때문에 H_LL이 block diagonal이다.**

### Trade-off

Landmark를 제거하면 state dimension은 크게 줄어든다. 하지만 landmark가 여러 pose를 함께 관측했다면 그 정보가 pose-pose coupling으로 바뀌므로 fill-in이 생기고 pose-only reduced system은 더 dense해질 수 있다.

즉

`variable 수 감소 ↔ sparsity 악화 가능`

이라는 trade-off가 있으며, 이것이 이후 elimination ordering 문제로 연결된다.

---

## 8. Marginalization — 확률 관점에서 Variable의 정보를 보존하며 제거

Probability 관점에서 joint distribution `p(x,y)`에서 `y`를 제거하고 `x`만 남기려면

`p(x) = ∫ p(x,y) dy`

를 계산한다.

이것이 marginalization이다.

Optimization 관점에서는 variable elimination과 Schur Complement를 보았고, probability 관점에서는 marginalization을 본 것이다.

Gaussian / linearized estimation problem에서는 이 두 관점이 밀접하게 연결된다.

```text
Optimization 관점:
Variable Elimination
→ Schur Complement

Probability 관점:
Joint Gaussian
→ Marginalization
→ Remaining variables의 marginal distribution
```

핵심 원리는 동일하다.

> **제거할 variable 자체는 더 이상 유지하지 않지만, 그 variable과 관련된 uncertainty/correlation/information은 남은 state에 보존한다.**

---

## 9. Sliding-window VIO에서 왜 Marginalization이 필요한가?

시간이 흐를수록 VIO state를 계속 추가하면

```text
x1 → x2 → x3 → x4 → x5 → x6 → ...
```

state dimension과 계산량이 계속 증가한다.

Sliding-window estimator는 최근 일정 개수의 state만 유지한다.

예:

```text
기존 window: x1 x2 x3 x4 x5
새 state x6 추가
→ x1 marginalize
→ 새 window: x2 x3 x4 x5 x6
```

`x1`을 단순 삭제하면 `x1` 및 과거 measurement들이 현재 state에 제공하던 정보까지 사라진다.

따라서 오래된 state를 marginalize하여 그 정보를 **marginalization prior / prior factor** 형태로 남은 state에 전달한다.

```text
old state + old measurements
→ marginalization
→ compressed prior information
→ remaining window states
```

이는 recursive estimation 관점에서 과거 posterior의 정보를 다음 추정의 prior로 이어가는 것과 연결해 이해할 수 있다.

---

## 10. Sliding Window의 장점과 대가

### 장점 — bounded computation

Window 크기를 고정하면 최적화 state dimension이 bounded된다.

따라서 메모리 사용량과 optimization problem의 크기를 제한할 수 있어 real-time VIO에 유리하다.

### 대가 1 — Marginalization / Linearization Error

SLAM/VIO는 nonlinear system이다.

Marginalization 시점에 nonlinear factor를 특정 estimate 주변에서 linearize하고 Schur Complement를 통해 prior로 압축한다.

```text
원래 nonlinear factors
→ 특정 X_k에서 linearization
→ marginalization
→ linearized prior
```

나중에 현재 estimate가 변해도 이미 제거된 과거 state와 원래 nonlinear factor를 다시 꺼내 새로운 point에서 재선형화할 수 없다.

따라서 full-batch optimization과 완전히 동일하지 않으며, marginalization 당시의 linearization point에 정보가 고정되는 효과가 생긴다.

### 대가 2 — Local window 밖의 state를 직접 joint optimization하지 못함

순수 sliding-window estimator 내부에서는 marginalize된 과거 state를 다시 현재 window에 포함하여 full-batch처럼 재최적화할 수 없다.

하지만 이것이 **SLAM 시스템 전체에서 loop closure가 불가능하다는 뜻은 아니다.**

실제 시스템은 다음처럼 구성할 수 있다.

```text
Local estimator
→ Sliding-window VIO
→ 실시간 local state estimation

Global backend
→ Keyframe / Pose Graph 유지
→ Loop Closure
→ Global correction
```

즉 local estimator는 marginalization으로 bounded computation을 확보하고, 별도의 global backend가 과거 keyframe과 loop closure를 유지할 수 있다.

---

## 11. 오늘의 오개념 / 정확한 표현

### `Schur Complement로 landmark를 버린다`

아니다.

> Landmark variable은 제거하지만 landmark가 pose에 제공한 information은 reduced pose system에 남긴다.

### `Landmark끼리는 원래 독립이라 H_LL은 항상 diagonal이다`

정확히는 전형적인 BA에서 landmark-landmark factor가 없기 때문에 **block diagonal**이다. Factor graph가 달라지면 이 구조도 달라질 수 있다.

### `Marginalization은 과거 state를 삭제하는 것`

단순 삭제와 다르다.

> 과거 state는 제거하지만 과거 measurement가 남은 state에 제공하던 information을 prior로 보존한다.

### `Sliding-window VIO는 loop closure를 할 수 없다`

Local window 자체는 marginalize된 모든 과거 state를 다시 full-batch optimization할 수 없다. 그러나 별도의 global pose-graph backend를 두면 loop closure와 global correction이 가능하다.

---

# Final Mental Model

```text
Factor Graph
↓
각 measurement는 일부 variable만 제약
↓
Sparse Jacobian
↓
Sparse Hessian / Information Matrix
↓
Sparse optimization으로 큰 SLAM 문제를 계산 가능
↓
하지만 state가 너무 많으면 일부 variable을 제거하고 싶음
↓
그냥 삭제하면 정보 손실
↓
Schur Complement
= variable을 제거하면서 그 정보는 reduced system에 흡수
↓
공통 variable을 제거하면 주변 variable 사이에 fill-in 발생
↓
BA에서는 H_LL의 block-diagonal 구조를 이용해 landmark를 효율적으로 제거
↓
VIO에서는 시간에 따라 과거 state가 계속 쌓임
↓
Sliding Window
↓
오래된 state를 Marginalization
↓
과거 information을 prior로 남김
↓
bounded computation 확보
↓
대신 과거 nonlinear factor의 재선형화가 불가능하여 full-batch와 차이가 발생
↓
필요하면 별도의 global pose graph/backend가 loop closure와 global correction 담당
```

## One-line memory

> **Factor Graph의 local connectivity가 sparse Jacobian/Hessian을 만들고, Schur Complement는 불필요한 variable을 제거하면서 그 정보를 남은 state에 흡수한다. Bundle Adjustment는 이 구조로 landmark를 효율적으로 제거하고, sliding-window VIO는 marginalization으로 과거 정보를 prior에 보존하면서 계산량을 제한한다.**

---

## Next

이번 memo의 학습 범위는 여기까지 완료.

다음 학습에서는 새로운 backend 최적화 기법을 바로 늘리기보다 전체 SLAM roadmap에 따라 다음 prerequisite/주제로 진행한다. `Bayes tree`, `iSAM2`, detailed elimination ordering, FEJ 등의 세부 주제는 실제 필요성이 등장할 때 학습한다.
