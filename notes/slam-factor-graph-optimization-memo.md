# SLAM Factor Graph — Optimization / Sparsity Memo

> 이번 세션에서 새로 학습한 내용을 기존 `slam-factor-graph-memo.md`와 분리해 기록한다. 목적은 **Factor Graph가 어떻게 실제 nonlinear optimization과 sparse computation으로 연결되는지**를 나중에 다시 복원하는 것.

## Storyline

```text
Factor Graph
→ 각 factor는 일부 variable만 본다
→ residual을 state에 대해 미분하면 Jacobian
→ 연결되지 않은 variable에 대한 derivative는 0
→ 따라서 Jacobian이 sparse해진다
→ Gauss-Newton에서 H ≈ JᵀΩJ
→ Hessian / information matrix도 graph connectivity를 반영해 sparse해진다
→ SLAM은 state가 매우 커도 factor가 local하게 연결되므로 sparse optimization이 가능하다
→ 그런데 variable을 제거하면 주변 variable 사이에 새로운 effective coupling이 생길 수 있다
→ fill-in 등장
→ variable elimination / Schur complement로 연결
```

---

## 1. Nonlinear Least Squares → Linearization

SLAM의 전체 cost는:

`J_cost(X) = Σ_i r_i(X)^T Ω_i r_i(X)`

이다.

Residual이 nonlinear이면 현재 estimate `X_k` 주변에서 작은 update `ΔX`에 대해:

`r(X_k + ΔX) ≈ r(X_k) + J ΔX`

로 local linearization한다.

핵심은:

> **nonlinear 문제를 현재 추정값 주변의 local linear problem으로 바꾸어 반복적으로 풀기 위해 linearization한다.**

근사는 현재 점 주변에서만 잘 맞으므로:

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

Gauss-Newton은 이 과정에서 사용하는 대표적인 방법이며, 별도의 새로운 알고리즘으로 깊게 다루기보다 이미 알고 있는 최적화 지식을 SLAM factor 구조에 연결한다.

---

## 2. Jacobian = residual sensitivity

Jacobian은 단순히 "편미분한 행렬"로 외우기보다:

> **state를 조금 움직였을 때 residual이 얼마나, 어느 방향으로 변하는지를 나타내는 local sensitivity map**

으로 이해한다.

간단한 예:

`r = 5 - (x2 - x1) = 5 - x2 + x1`

따라서:

`∂r/∂x1 = +1`

`∂r/∂x2 = -1`

즉 `x1`을 +1 움직이면 residual은 +1 변하고, `x2`를 +1 움직이면 residual은 -1 변한다.

일반적으로:

`Δr ≈ J ΔX`

이다.

---

## 3. Factor Graph → Sparse Jacobian

예를 들어:

```text
x1 ── x2 ── x3
```

두 factor가:

`r12 = r12(x1,x2)`

`r23 = r23(x2,x3)`

라고 하자.

전체 Jacobian의 구조는:

```text
J = [ *  *  0
      0  *  * ]
```

이다.

왜냐하면:

`∂r12/∂x3 = 0`

`∂r23/∂x1 = 0`

이기 때문이다.

즉 **전혀 관계없는 variable에 대한 residual의 편미분은 0**이다.

따라서:

`Factor Graph의 local connectivity`

`→ residual dependency`

`→ Jacobian의 non-zero structure`

`→ sparse Jacobian`

이라는 연결이 생긴다.

### 핵심 기억

Jacobian의 sparse structure는 우연히 생기는 것이 아니라 **Factor Graph가 어떤 variable을 어떤 factor로 연결했는지에 의해 결정된다.**

---

## 4. Jacobian → Hessian / Information Matrix

Gauss-Newton에서는 linearized weighted least squares를 풀어:

`H ΔX = -g`

형태의 선형 시스템을 얻는다.

개념적으로:

`H ≈ Jᵀ Ω J`

이다.

여기서:

- `J`: residual sensitivity
- `Ω = Σ⁻¹`: measurement information / confidence
- `H`: Gauss-Newton Hessian approximation이며 information structure를 나타냄
- `g`: 현재 residual이 만드는 gradient 방향
- `ΔX`: state correction

예를 들어 `x1 -- x2 -- x3` 구조라면 `H`도 개념적으로:

```text
H ≈ [ *  *  0
      *  *  *
      0  *  * ]
```

같은 구조를 가진다.

`H_13 = 0`이라는 것은 단순히 행렬 계산 결과가 아니라:

> **현재 factor 구조에서 x1과 x3 사이에 직접적인 measurement coupling이 없다.**

는 의미로 해석할 수 있다.

따라서:

`Factor Graph connectivity`

`→ Jacobian sparsity`

`→ Hessian / information matrix sparsity`

가 연결된다.

---

## 5. 왜 Sparse Structure가 실제 SLAM에서 중요한가?

실제 SLAM에서는 pose가 수천~수만 개가 될 수 있다.

전체 state가 매우 커져도 하나의 measurement factor는 보통 소수의 variable만 연결한다.

따라서 전체 Jacobian/Hessian은:

> **크기는 매우 크지만 대부분이 0인 sparse matrix**

가 된다.

이 sparse structure를 이용하면 거대한 dense matrix를 그대로 계산하는 것보다 훨씬 효율적으로 optimization을 수행할 수 있다.

따라서 Factor Graph는 단순한 시각화 방법이 아니라:

> **대규모 SLAM optimization의 계산 구조를 드러내는 표현**

이라고 볼 수 있다.

---

## 6. Variable Elimination → Fill-in

다음 구조를 생각한다.

```text
x1 ── x2 ── x3
```

`x2`는 두 factor에 동시에 참여한다:

`f12(x1,x2)`

`f23(x2,x3)`

따라서 `x2`를 optimization에서 제거(eliminate)하면, `x2`가 가지고 있던 두 관계의 효과를 `x1`과 `x3` 사이의 새로운 effective constraint로 표현할 수 있다.

```text
제거 전:

x1 ── x2 ── x3

제거 후:

x1 ───────── x3
```

이처럼 elimination 과정에서 원래 없던 non-zero coupling이 생기는 것을 **fill-in**이라고 한다.

### 중요한 오개념

fill-in은 단순히 `x1 → x2 → x3`가 시간 순서로 연속되기 때문에 생기는 것이 아니다.

핵심은:

> **제거되는 variable이 여러 factor를 통해 주변 variable들을 연결하고 있었기 때문이다.**

같은 현상은 landmark에서도 발생할 수 있다:

```text
x1 ── L ── x2
```

landmark `L`을 제거하면 `x1`과 `x2` 사이에 effective coupling이 생길 수 있다.

---

## 7. 다음 개념으로 이어지는 이유

여기까지 오면 새로운 문제가 생긴다.

SLAM에는 pose와 landmark가 매우 많다. 특히 landmark가 많으면 전체 optimization system이 커진다.

그렇다면:

> **어떤 variable을 먼저 제거하면 계산을 더 효율적으로 만들 수 있을까?**

라는 문제가 자연스럽게 등장한다.

이 질문이 다음 학습의 출발점이다.

```text
Variable Elimination
→ Schur Complement
→ 왜 landmark를 먼저 제거하는가?
→ Bundle Adjustment의 구조
→ VIO에서 marginalization은 무엇인가?
```

Schur Complement와 marginalization 자체는 아직 본격적으로 학습하지 않았으므로 이 memo에서는 여기서 멈춘다.

---

## One-line memory

> **Factor Graph의 연결 구조가 Jacobian의 sparsity를 만들고, 그 구조가 Hessian/information matrix에도 이어지기 때문에 대규모 SLAM을 sparse optimization으로 풀 수 있다. Variable을 제거하면 fill-in이 생길 수 있으므로 이후에는 elimination 순서와 Schur Complement가 필요해진다.**
