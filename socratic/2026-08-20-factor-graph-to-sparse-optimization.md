# 2026-08-20 — Factor Graph → Nonlinear Optimization → Sparse Structure

## 오늘의 핵심 Story

SLAM을 처음 보면 `Factor Graph`는 단순히 pose와 measurement를 연결해 놓은 그림처럼 보인다. 하지만 실제로는 이 연결 구조가 **최적화 문제의 계산 구조 자체를 결정한다.**

```text
Odometry
→ drift
→ loop closure로 장거리 constraint 추가
→ pose graph
→ landmark도 unknown이므로 factor graph
→ sensor noise를 고려한 probabilistic constraint
→ covariance / information
→ MLE / MAP
→ weighted nonlinear least squares
→ joint optimization
→ nonlinear residual을 local linearization
→ Jacobian
→ Gauss-Newton
→ Jacobian의 sparsity
→ Hessian / information matrix의 sparsity
→ sparse optimization
```

이 흐름을 기억하면 Factor Graph가 왜 등장했고, Jacobian/Hessian을 왜 공부하는지 다시 복원할 수 있다.

---

## 1. Factor Graph: measurement가 특정 variable들을 제약한다

Factor graph에서 variable은 pose, landmark, bias 등 추정해야 할 unknown state가 될 수 있다.

예:

```text
x1 -- x2 -- x3
```

`f12`는 `x1, x2`에 의존하고 `f23`은 `x2, x3`에 의존한다.

```text
r12 = r12(x1, x2)
r23 = r23(x2, x3)
```

Factor는 단순한 "두 state 사이의 상관관계"가 아니라, **하나의 measurement가 특정 variable들의 값을 동시에 제약하는 probabilistic/geometric constraint**다.

---

## 2. Joint optimization: 모든 factor를 동시에 설명하는 state를 찾는다

각 measurement의 residual을

```text
r_i(X) = z_i - h_i(X)
```

라고 하면 전체 목적함수는 Gaussian noise 가정에서

```text
J(X) = Σ r_i(X)^T Ω_i r_i(X)
```

가 된다.

따라서 optimizer는 특정 pose 하나를 맞추는 것이 아니라 **전체 measurement set을 가장 잘 설명하는 하나의 joint state configuration**을 찾는다.

Loop closure가 추가되면 기존 odometry trajectory와 새로운 long-range constraint를 동시에 만족시켜야 하므로 여러 pose가 함께 수정될 수 있다.

---

## 3. Nonlinear least squares → local linearization

SLAM의 residual은 SE(2)/SE(3) geometry 등으로 인해 일반적으로 nonlinear하다.

현재 estimate를 `X_k`라 하고 작은 state update를 `ΔX`라 하면,

```text
r(X_k + ΔX) ≈ r(X_k) + J ΔX
```

로 local linearization한다.

중요한 이유는 "미분이 너무 복잡해서"가 아니다.

> 현재 추정값 주변의 nonlinear 문제를 local linear problem으로 바꾸어 반복적으로 풀 수 있기 때문이다.

단, linearization은 현재 점 주변에서만 정확하므로 iterative optimization이 필요하다.

```text
현재 X_k
→ linearize
→ J 계산
→ linear problem 해결
→ ΔX 계산
→ X_{k+1} = X_k + ΔX
→ 다시 linearize
→ 반복
```

---

## 4. Jacobian의 robotics 의미

Jacobian은 단순한 "편미분 행렬"이 아니라,

> **각 state를 조금 움직였을 때 특정 measurement residual이 얼마나, 어느 방향으로 변하는지를 나타내는 local sensitivity map**

이다.

간단한 1D odometry 예:

```text
r(x1,x2) = 5 - (x2 - x1)
         = 5 - x2 + x1
```

따라서

```text
∂r/∂x1 = +1
∂r/∂x2 = -1
```

이고

```text
J = [1  -1]
```

이다.

즉 `x1`을 +1 움직이면 residual은 +1 변하고, `x2`를 +1 움직이면 residual은 -1 변한다.

일반적으로

```text
Δr ≈ J ΔX
```

이며, 이것이 nonlinear optimization에서 state update와 residual change를 연결한다.

---

## 5. 여러 factor를 합치면 Jacobian이 sparse해진다

예를 들어

```text
x1 -- x2 -- x3
```

이고

```text
r12 = r12(x1,x2)
r23 = r23(x2,x3)
```

이면 전체 Jacobian은 구조적으로

```text
J = [ *  *  0
      0  *  * ]
```

가 된다.

왜냐하면 `r23`은 `x1`과 아무 관계가 없으므로

```text
∂r23/∂x1 = 0
```

이고, `r12`는 `x3`와 관계가 없으므로

```text
∂r12/∂x3 = 0
```

이다.

따라서

```text
Factor Graph의 local connectivity
→ Jacobian의 non-zero structure
→ Jacobian sparsity
```

가 된다.

이것은 우연한 행렬 특성이 아니라 **Factor Graph의 연결 구조가 수학적 최적화 구조에 직접 반영된 결과**다.

---

## 6. Gauss-Newton과 Hessian / Information Matrix

Gauss-Newton에서는 linearized weighted least squares를 풀고, 일반적으로

```text
H ΔX = -g
```

형태의 선형 시스템을 얻는다.

여기서 개념적으로

- `J_i`: state 변화가 measurement residual에 미치는 sensitivity
- `Ω_i = Σ_i^{-1}`: measurement confidence / information
- `H ≈ J^T Ω J`: Gauss-Newton Hessian approximation, information structure
- `g`: 현재 residual이 만드는 gradient 방향
- `ΔX`: state correction

이다.

중요한 것은 Hessian도 Factor Graph의 connectivity를 반영한다는 것이다.

예를 들어

```text
x1 -- x2 -- x3
```

이면 구조적으로

```text
H ≈ [ *  *  0
      *  *  *
      0  *  * ]
```

같은 형태가 나타난다.

`H_13`이 0인 직관적 이유는 `x1`과 `x3`가 하나의 factor에서 직접 함께 제약되지 않기 때문이다.

따라서

```text
Factor Graph connectivity
→ Jacobian sparsity
→ Hessian / information matrix sparsity
```

라는 연결이 성립한다.

---

## 7. 왜 sparsity가 SLAM에서 중요한가?

실제 SLAM에서는 pose가 수천~수만 개가 될 수 있다.

전체 state가 커지더라도 각 measurement는 보통 소수의 variable만 연결한다.

따라서 Jacobian/Hessian은 **크기는 매우 크지만 대부분이 0인 sparse matrix**가 된다.

이 sparse structure를 이용하면 거대한 dense matrix를 그대로 계산하는 것보다 훨씬 효율적으로 optimization을 수행할 수 있다.

즉 Factor Graph는 단순한 visualization이 아니라,

> **대규모 SLAM optimization을 계산 가능하게 만드는 구조적 정보를 제공한다.**

---

## 8. Variable elimination과 fill-in의 직관

다음 구조에서

```text
x1 -- x2 -- x3
```

`x2`는 `x1`과 연결된 factor와 `x3`와 연결된 factor에 동시에 참여한다.

따라서 `x2`를 optimization에서 제거/eliminate하면 `x2`가 가지고 있던 두 관계의 효과를 `x1`과 `x3` 사이의 새로운 effective constraint로 표현할 수 있다.

```text
제거 전:
x1 -- x2 -- x3

제거 후:
x1 -------- x3
```

이처럼 elimination 과정에서 원래 없던 non-zero coupling이 생기는 것을 **fill-in**이라고 한다.

중요한 오개념:

> fill-in은 단순히 pose가 시간 순서로 연속되어 있기 때문에 발생하는 것이 아니다.

핵심은 **제거되는 variable이 여러 factor를 통해 주변 variable들을 연결하고 있었기 때문**이다.

같은 현상은 landmark에서도 발생한다.

```text
x1 -- L -- x2
```

에서 landmark `L`을 제거하면 `x1`과 `x2` 사이에 effective coupling이 생길 수 있다.

이것이 이후 **variable elimination → Schur complement → landmark elimination → Bundle Adjustment / VIO marginalization**으로 이어진다.

---

## 9. 현재까지의 핵심 기억 문장

### 한 문장

> **Factor Graph의 연결 구조가 residual의 dependency를 결정하고, 그 dependency가 Jacobian/Hessian의 sparsity를 결정하며, 이 sparsity를 이용하기 때문에 대규모 SLAM optimization이 실제로 가능해진다.**

### Story로 기억하기

```text
Odometry만 쓰면 drift가 생긴다.
→ loop closure라는 새로운 constraint가 필요하다.
→ pose와 landmark가 모두 unknown이므로 factor graph로 전체 문제를 표현한다.
→ 센서가 noisy하므로 각 constraint에는 covariance/information이 붙는다.
→ Bayesian/MLE/MAP 관점에서 weighted nonlinear least squares가 된다.
→ nonlinear이므로 현재 estimate 주변에서 linearize한다.
→ Jacobian은 state 변화가 residual에 미치는 sensitivity를 나타낸다.
→ 각 factor는 일부 variable만 보므로 Jacobian이 sparse하다.
→ H ≈ JᵀΩJ도 sparse한 구조를 가진다.
→ 이 sparsity를 이용하면 거대한 SLAM 문제를 효율적으로 풀 수 있다.
→ 그런데 variable을 제거하면 fill-in이 생길 수 있다.
→ 따라서 어떤 variable을 어떤 순서로 제거할지가 중요해진다.
→ 여기서 variable elimination / Schur complement가 등장한다.
```

---

## 10. 오늘의 오개념 교정

### `factor = 두 state의 상관관계`
정확히는 **measurement가 관련 variable들을 동시에 제약하는 probabilistic/geometric factor**다.

### `linearization은 nonlinear 미분이 너무 복잡해서 한다`
핵심 이유는 **현재 estimate 주변에서 nonlinear problem을 local linear problem으로 바꾸어 반복적으로 풀기 위해서**다.

### `fill-in은 시간 순서 때문에 생긴다`
아니다. **eliminate되는 variable이 여러 factor를 통해 주변 variable들을 연결하고 있기 때문에** 생긴다.

---

## Next

다음 학습 주제:

```text
Variable Elimination
→ Schur Complement
→ 왜 landmark를 먼저 제거하는가?
→ Bundle Adjustment에서의 구조
→ VIO에서 marginalization이 왜 필요한가?
```

수식 유도보다 먼저 `"왜 landmark를 제거하고 싶은가?"`라는 시스템 관점에서 시작한다.
