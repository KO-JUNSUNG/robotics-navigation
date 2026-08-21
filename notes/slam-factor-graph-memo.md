# SLAM / Factor Graph Memo — Storyline

> 목적: 공식을 암기하는 것이 아니라 **왜 다음 개념이 필요해졌는지**를 다시 떠올리기 위한 복습용 memo.

## 0. 출발점 — Odometry만으로 robot trajectory를 만들면?

Robot pose를 `T_WRi`라고 하고, odometry가 `Ri → Ri+1`의 상대 이동을 측정하면:

`T_WR(i+1) = T_WRi T_RiR(i+1)`

따라서 odometry를 계속 composition하면 trajectory를 만들 수 있다.

하지만 각 measurement에 작은 오차가 있으므로 trajectory가 길어질수록 drift가 누적된다.

**문제:** 한 방향으로 계속 누적되는 relative measurement만으로는 trajectory를 global하게 교정할 방법이 부족하다.

---

## 1. Loop closure — 과거 장소를 다시 보면 무엇이 달라지는가?

R1에서 본 landmark `L`을 R5에서도 다시 봤다고 하자.

같은 물체라면 두 관측이 같은 world point를 설명해야 한다:

`T_WR1 ^R1 p_L ≈ T_WR5 ^R5 p_L`

즉 R1과 R5 사이에 **새로운 long-range constraint**가 생긴다.

이를 landmark constraint로 쓸 수도 있고, scan matching 등으로 상대 pose를 직접 추정하여:

`T_R1R5^loop ≈ T_WR1^{-1} T_WR5`

라는 loop-closure relative-pose constraint로 만들 수도 있다.

### 왜 중요한가?

Odometry가 만든 긴 chain에 R1 ↔ R5라는 shortcut이 생긴다.

`R1 ─ R2 ─ R3 ─ R4 ─ R5`

`└──────── loop ────────┘`

이제 optimizer는 기존 odometry와 loop closure를 동시에 만족하도록 전체 trajectory를 조정할 수 있다. 따라서 누적 drift가 trajectory 전체에 걸쳐 보정될 수 있다.

**주의:** ICP는 주로 두 scan을 registration하여 상대 pose를 추정하는 도구다. "같은 장소인가?"를 찾는 place recognition과 scan registration은 구분한다.

---

## 2. Constraint를 어떻게 평가할까? — Residual

Odometry measurement를 `z_12`라고 하자.

현재 추정한 두 pose가 만들어내는 예상 relative pose는:

`h(T_WR1,T_WR2) = T_WR1^{-1} T_WR2`

따라서 기본적인 measurement-model 관점에서는:

`r_12 = z_12 - h(X)`

즉 **측정값과 현재 state가 예측하는 measurement의 차이**가 residual이다.

SLAM의 각 edge/factor는 결국:

`measurement → predicted measurement → residual`

이라는 구조를 가진다.

**주의:** SE(2)/SE(3) pose는 일반 Euclidean vector가 아니므로 실제 구현에서는 transformation matrix를 단순히 빼지 않고 relative transform의 Lie-algebra/local-vector representation 등을 사용한다.

---

## 3. Pose Graph — 왜 Graph가 필요한가?

R1~R5를 각각 독립적으로 추정하는 것이 아니라, **pose들을 node로 보고 measurement를 edge로 연결**하면 문제 구조가 보인다.

`R1 ──odom── R2 ──odom── R3 ──odom── R4 ──odom── R5`

Loop closure가 추가되면:

`R1 ─────────loop──────── R5`

즉:

- **node:** robot pose
- **edge:** 두 pose 사이의 relative measurement/constraint

이 구조를 **pose graph**라고 한다.

### 왜 유용한가?

우리가 풀고 싶은 것은 모든 pose를 동시에 조정하는 것:

`X = {T_WR1,...,T_WRN}`

각 edge는 residual `r_i(X)`를 만들고, 전체 trajectory는 모든 edge를 동시에 만족하도록 최적화된다.

---

## 4. 그런데 실제 SLAM에는 pose만 있는 게 아니다 — Landmark가 unknown이다

SLAM에서는 robot pose뿐 아니라 landmark 위치도 모른다.

`x_i = robot pose`

`l_j = landmark`

landmark observation은:

`z_ij = h(x_i,l_j) + v`

즉 하나의 observation은 **pose와 landmark를 함께 연결하는 constraint**다.

### Pose Graph → Factor Graph

Pose graph는 pose만 unknown으로 두는 경우를 표현하기 좋다.

하지만 SLAM에서는 landmark도 unknown이다. 따라서 graph를 **state variable과 measurement factor의 관계**로 표현한다.

예:

`z_12 → (x1,x2)`  : odometry factor

`z_3L → (x3,L)`    : landmark observation factor

- **variable node:** pose, landmark 등 우리가 추정할 unknown
- **factor:** 특정 variable들을 직접 연결하는 measurement/probabilistic constraint

이것이 **factor graph**다.

Pose graph는 pose만 variable로 두는 factor graph의 특수한 경우로 볼 수 있다.

**주의:** 같은 landmark를 반복 관측하는 것 자체를 모두 loop closure라고 부르지는 않는다. Loop closure는 보통 시간적으로 떨어진 과거 장소/pose와 현재 pose를 연결하는 장거리 constraint를 의미한다.

---

## 5. Factor는 왜 확률이 되는가? — 센서는 noisy하다

실제 measurement는 정확한 equality constraint가 아니다:

`z_i = h_i(X) + v_i`

Gaussian noise를 가정하면:

`z_i ~ N(h_i(X), Σ_i)`

따라서 각 factor는 단순한 선이 아니라 **현재 state가 measurement를 얼마나 잘 설명하는지에 대한 likelihood**를 제공한다.

전체 measurement가 조건부 독립이면:

`p(Z|X) = ∏_i p(z_i|X_i)`

이처럼 전체 likelihood를 여러 local factor로 분해할 수 있다.

---

## 6. Bayesian → MLE/MAP → Weighted Nonlinear Least Squares

우리가 원하는 것은 state의 posterior:

`p(X|Z) ∝ p(Z|X)p(X)`

- `p(Z|X)`: sensor measurement가 현재 state에서 얼마나 그럴듯한가
- `p(X)`: prior

Gaussian likelihood를 사용하면 negative log-likelihood가 residual의 quadratic form이 된다:

`r_i^T Σ_i^{-1} r_i`

따라서 MLE/MAP estimation은 다음 optimization으로 연결된다:

`X* = argmin_X Σ_i r_i^T Σ_i^{-1} r_i`

즉:

`Gaussian → likelihood → MLE/MAP → Weighted Nonlinear Least Squares`

여기서:

- `Σ_i`: covariance / uncertainty
- `Σ_i^{-1}`: information matrix / weight
- 작은 uncertainty → 큰 weight → 더 강한 constraint

---

## 7. Joint optimization — 왜 constraint 하나가 전체 trajectory를 바꾸는가?

최적화 대상은 특정 pose 하나가 아니라 전체 state:

`X = {x1,x2,...,xN,L1,L2,...}`

목적함수는 모든 factor의 cost를 합친다:

`J(X) = Σ_i r_i(X)^T Ω_i r_i(X)`

따라서 optimizer는 한 constraint만 만족시키는 것이 아니라 **전체 constraint의 cost가 가장 작아지는 joint solution**을 찾는다.

오차가 반드시 균등하게 분산되는 것은 아니다. 각 factor의 covariance/information에 따라 신뢰도가 높은 constraint를 더 강하게 만족시키는 방향으로 solution이 정해진다.

---

## 8. Nonlinear optimization — 왜 Jacobian과 linearization이 필요한가?

SLAM의 residual은 SE(2)/SE(3) geometry 등으로 인해 일반적으로 nonlinear이다.

현재 estimate를 `X_k`라 하고 작은 state update를 `ΔX`라 하면:

`r(X_k + ΔX) ≈ r(X_k) + J ΔX`

로 local linearization한다.

핵심은 **nonlinear 문제를 현재 estimate 주변의 local linear problem으로 바꾸어 반복적으로 풀 수 있게 하는 것**이다.

Linearization은 현재 점 주변에서만 잘 맞으므로 iterative optimization이 필요하다:

`현재 X_k → linearize → J 계산 → linear problem 해결 → ΔX 계산 → X_{k+1} = X_k + ΔX → 다시 linearize → 반복`

Gauss-Newton은 이 과정에서 사용하는 대표적인 방법이며, 이미 알고 있는 최적화 지식과 SLAM의 factor 구조를 연결하는 역할로 이해한다.

---

## 9. Jacobian — residual sensitivity

Jacobian은 단순한 편미분 행렬이 아니라,

> **각 state를 조금 움직였을 때 특정 measurement residual이 얼마나, 어느 방향으로 변하는지를 나타내는 local sensitivity map**

이다.

간단한 예:

`r(x1,x2) = 5 - (x2 - x1) = 5 - x2 + x1`

따라서:

`∂r/∂x1 = +1`

`∂r/∂x2 = -1`

즉 `x1`을 +1 움직이면 residual은 +1 변하고, `x2`를 +1 움직이면 residual은 -1 변한다.

일반적으로:

`Δr ≈ J ΔX`

이며, 이것이 state update와 residual change를 연결한다.

---

## 10. Factor Graph → Sparse Jacobian

이제 factor graph의 연결 구조가 계산 구조로 이어진다.

예:

`x1 ── x2 ── x3`

factor가:

`r12 = r12(x1,x2)`

`r23 = r23(x2,x3)`

라면 전체 Jacobian은 구조적으로:

`J = [ *  *  0`
`      0  *  * ]`

이다.

왜냐하면 `r23`은 `x1`과 아무 관계가 없으므로:

`∂r23/∂x1 = 0`

이고 `r12`는 `x3`와 관계가 없으므로:

`∂r12/∂x3 = 0`

이다.

즉:

`Factor Graph의 local connectivity`

`→ Jacobian의 non-zero structure`

`→ Jacobian sparsity`

가 된다.

**중요:** 이 sparsity는 우연히 생기는 것이 아니라 factor가 어떤 variable에 의존하는지가 그대로 반영된 결과다.

---

## 11. Sparse Jacobian → Sparse Hessian / Information Matrix

Gauss-Newton에서 linearized problem을 풀면 보통:

`H ΔX = -g`

형태의 선형 시스템이 나오고,

`H ≈ J^T Ω J`

이다.

여기서 `H`는 Gauss-Newton Hessian approximation으로 볼 수 있으며 information structure와 밀접하게 연결된다.

예를 들어:

`x1 ── x2 ── x3`

이면 구조적으로:

`H ≈ [ *  *  0`
`      *  *  *`
`      0  *  * ]`

같은 형태가 나타난다.

`x1`과 `x3`가 하나의 factor에서 직접 함께 제약되지 않기 때문에 해당 coupling이 직접적으로 생기지 않는다는 것이 핵심 직관이다.

따라서:

`Factor Graph connectivity`

`→ Jacobian sparsity`

`→ Hessian / information matrix sparsity`

라는 연결이 성립한다.

---

## 12. 왜 sparsity가 실제 SLAM에서 중요한가?

실제 SLAM에서는 pose가 수천~수만 개가 될 수 있다.

하지만 각 measurement는 보통 소수의 variable만 연결한다.

따라서 전체 Jacobian/Hessian은 **크기는 매우 크지만 대부분이 0인 sparse matrix**가 된다.

이 sparse structure를 이용하면 거대한 dense matrix를 그대로 다루는 것보다 훨씬 효율적으로 optimization을 수행할 수 있다.

즉 Factor Graph는 단순한 visualization이 아니다.

> **Factor Graph의 connectivity는 대규모 SLAM optimization의 계산 구조를 제공하고, 그 sparsity를 이용하기 때문에 큰 문제를 실제로 풀 수 있다.**

---

## 13. Variable Elimination → Fill-in의 첫 직관

다음 구조를 보자:

`x1 ── x2 ── x3`

`x2`는 `x1`과 연결된 factor와 `x3`와 연결된 factor에 **동시에 참여하는 중간 variable**이다.

따라서 `x2`를 eliminate하면 `x2`가 가지고 있던 두 관계의 효과를 `x1`과 `x3` 사이의 effective constraint로 표현할 수 있다.

개념적으로:

`제거 전: x1 -- x2 -- x3`

`제거 후: x1 -------- x3`

원래 graph에 없던 coupling이 elimination 과정에서 새로 나타나는 것을 **fill-in**이라고 한다.

**중요한 오개념:** fill-in은 단순히 pose가 시간 순서로 연속되어 있기 때문에 생기는 것이 아니다. 핵심은 **제거되는 variable이 여러 factor를 통해 주변 variable들을 연결하고 있었기 때문**이다.

같은 현상은 landmark에서도 나타날 수 있다:

`x1 -- L -- x2`

에서 landmark `L`을 제거하면 `x1`과 `x2` 사이에 effective coupling이 생길 수 있다.

여기까지가 현재 학습한 범위다. **Schur complement와 marginalization은 아직 본격적으로 배우지 않았으므로 이 memo에서는 결론만 미리 적지 않는다.**

---

## 14. DOF / Observability와 Gauge Freedom

state가 가진 자유도보다 measurement가 제공하는 독립적인 constraint가 부족하면 state를 유일하게 결정할 수 없다.

Measurement model:

`z = h(X)`

local linearization:

`δz ≈ H δX`

`H = ∂h/∂X`

Jacobian의 rank는 locally 독립적인 constraint가 몇 개의 state 방향을 관측하는지와 관련된다.

`rank(H)` ↔ locally constrained directions

`null(H)` ↔ locally unobservable directions

SLAM에서는 relative measurement만으로 world frame의 absolute position/orientation이 결정되지 않는 gauge freedom이 대표적인 예다.

모든 pose에 동일한 rigid transform `G`를 적용해도:

`(G T_Wi)^{-1}(G T_Wj) = T_Wi^{-1}T_Wj`

이므로 모든 relative constraint가 그대로다.

따라서 loop closure가 있어도 gauge freedom은 사라지지 않는다.

보통 첫 pose를:

`T_WR1 = I`

로 고정하여 gauge를 제거한다. → **gauge fixing**

---

# Final mental model

처음 문제:

`odometry → trajectory → accumulated drift`

↓

과거 장소를 다시 봄:

`loop closure → 새로운 global constraint`

↓

pose를 node, constraint를 edge로 표현:

`pose graph`

↓

landmark도 unknown이므로 pose + landmark + measurement factor로 확장:

`factor graph`

↓

센서는 noisy하므로 constraint를 확률적으로 표현:

`likelihood / covariance`

↓

Bayesian estimation을 Gaussian assumption으로 전개:

`MLE/MAP → weighted nonlinear least squares`

↓

모든 factor를 동시에 최소화:

`joint optimization`

↓

nonlinear이므로 현재 estimate 주변에서 linearize:

`Jacobian → Gauss-Newton`

↓

각 factor는 일부 variable만 보므로:

`Sparse Jacobian → Sparse Hessian / Information Matrix`

↓

이 sparse structure를 이용해 대규모 문제를 효율적으로 풀 수 있음

↓

하지만 variable을 제거하면 주변 variable 사이에 새 coupling이 생길 수 있음:

`Variable elimination → fill-in`

↓

**다음:**

`Schur complement → landmark elimination → marginalization → Bundle Adjustment / VIO`

---

## 한 줄 요약

**SLAM은 noisy sensor가 제공하는 local relative constraints를 pose/landmark 사이의 factor로 연결하고, 그 모든 constraint를 동시에 가장 잘 만족하는 상태를 nonlinear optimization으로 추정하는 문제이며, Factor Graph의 local connectivity가 Jacobian/Hessian의 sparsity를 만들어 대규모 문제를 효율적으로 풀 수 있게 한다.**
