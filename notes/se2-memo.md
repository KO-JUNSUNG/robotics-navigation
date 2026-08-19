# SE(2) / Transformation Memo

> 짧게 복습하기 위한 이론 memo. 상세한 Socratic 대화는 `socratic/`에 보존한다.

## 1. Frame & pose

- `T_AB`는 **B frame → A frame** 변환.
- 점은 `^A p = T_AB ^B p`로 표현.
- Robot pose `T_WR`은 R frame이 W에 어떻게 놓여 있는지를 나타내며, 동시에 R의 좌표를 W로 옮기는 transformation.

## 2. 2D rotation

`R(theta) = [[cosθ, -sinθ], [sinθ, cosθ]]`

- `R^{-1} = R^T`
- 2D rotation은 `SO(2)`.

## 3. SE(2)

2D rigid-body transformation:

`T = [[R, t], [0, 1]]`

where `R ∈ SO(2)`, `t ∈ R²`.

Point를 homogeneous coordinate로 `[x, y, 1]^T`로 확장하면:

`^W p~ = T_WR ^R p~`

즉 `R p + t`를 하나의 matrix multiplication으로 표현할 수 있다.

Homogeneous coordinate의 핵심 장점은 **rotation + translation을 하나로 묶고 transformation composition을 행렬곱으로 표현**할 수 있다는 것.

## 4. Composition

`T_AB T_BC = T_AC`

읽는 순서:

`C → B → A`

따라서 robot이 `R1 → R2`로 이동했고 현재 pose가 `T_WR1`이라면:

`T_WR2 = T_WR1 T_R1R2`

즉 relative odometry는 기존 world pose의 오른쪽에 곱한다.

## 5. Inverse

`T = [[R,t],[0,1]]`이면

`T^{-1} = [[R^T, -R^T t],[0,1]]`

따라서:

`T_RW = T_WR^{-1}`

`^R p = T_RW ^W p`

## 6. Point vs vector

Point:

`^W p = R ^R p + t`

Vector/direction:

`^W v = R ^R v`

**translation은 vector에 더하지 않는다.**

## 7. State / measurement / measurement model

`X` = 추정하고 싶은 state.

`z` = 실제 sensor measurement.

`h(X)` = state가 X일 때 센서가 예상할 measurement.

Measurement model:

`z = h(X) + v`

Residual:

`r(X) = z - h(X)`

실제 SLAM에서는 현재 estimate `X_hat`을 넣어 `h(X_hat)`와 실제 `z`를 비교한다.

## 8. Odometry as a constraint

두 pose `T_WR1`, `T_WR2`가 있을 때 이들이 암시하는 relative pose는:

`T_R1R2 = T_WR1^{-1} T_WR2`

Wheel/LiDAR odometry measurement `z_12`는 이 relative transformation과 일치해야 한다는 constraint를 제공한다.

개념적으로:

`z_12 ≈ T_WR1^{-1} T_WR2`

즉 odometry는 단순히 "현재 위치"를 알려주는 것이 아니라 **두 pose 사이의 관계(relative-pose constraint)**를 제공한다.

## 9. SLAM으로 연결

Odometry를 계속 composition하면:

`T_WRN = T_WR1 T_R1R2 T_R2R3 ...`

작은 오차가 누적되어 drift가 발생한다.

SLAM은 odometry constraint뿐 아니라 landmark/LiDAR/camera 관측 등 여러 constraint를 함께 사용하여 trajectory와 map을 jointly estimate한다.

핵심 흐름:

`Sensor → measurement z → measurement model h(X) → residual r(X) → constraints → joint estimation`

## 10. 기억할 핵심 문장

- **Pose는 frame의 위치/자세를 나타내면서 동시에 frame-to-frame transformation으로 사용할 수 있다.**
- **`T_AB`: 오른쪽 B에서 출발해서 왼쪽 A로 간다.**
- **Composition은 frame 경로를 따라간다.**
- **Odometry는 relative-pose constraint다.**
- **SLAM에서는 pose와 map이 여러 sensor constraint를 동시에 만족하도록 추정된다.**
