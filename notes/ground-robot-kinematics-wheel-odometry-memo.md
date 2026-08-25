# Ground-Robot Kinematics and Wheel Odometry Memo

> 목적: differential-drive encoder에서 SE(2) pose increment가 만들어지는 과정, 그 과정이 의존하는 가정, 그리고 실제 SLAM estimator에서의 역할을 복원하기 위한 장기 개념 memo.

## Storyline

```text
좌우 wheel encoder
→ 각 바퀴의 이동거리
→ robot-frame linear velocity와 yaw rate
→ SE(2) pose integration
→ no-slip / nonholonomic assumptions
→ slip과 calibration error
→ odometry drift와 covariance propagation
→ IMU / LiDAR / SLAM constraint로 보정
```

## 1. Differential-drive body motion

왼쪽과 오른쪽 바퀴의 선속도를 각각 (v_L,v_R), 바퀴 사이 간격을 (b)라고 하면 robot 중심의 전진속도와 yaw 각속도는:

[
v=rac{v_L+v_R}{2}
]

[
omega=rac{v_R-v_L}{b}
]

이다.

- (v_R=v_L>0): 직진
- (v_R=-v_L): 중심을 기준으로 제자리 회전
- (v_R>v_L): 왼쪽 회전
- (v_L=0,v_R>0): 왼쪽 바퀴를 순간회전중심으로 회전

(omega)는 가속도의 미분이 아니라 방향각의 시간미분이다.

[
omega=dot{	heta}
]

## 2. Turning radius and instantaneous center of rotation

Robot 중심의 회전반경을 (R)이라 하면:

[
v=omega R
]

따라서:

[
R=rac{v}{omega}
=rac{b(v_L+v_R)}{2(v_R-v_L)}
]

좌우 바퀴의 원호 반지름은:

[
R_L=R-rac b2,qquad R_R=R+rac b2
]

이고:

[
v_L=omega R_L,qquad v_R=omega R_R
]

이다. 바깥 바퀴가 더 큰 반지름의 원호를 같은 각속도로 이동하므로 더 빠르게 움직여야 한다.

## 3. Continuous-time SE(2) motion model

World pose를 ((x,y,	heta))로 두고 (	heta)를 world (+x)축에서 반시계 방향으로 측정하면:

[
dot{x}=vcos	heta
]

[
dot{y}=vsin	heta
]

[
dot{	heta}=omega
]

여기서 (v)는 robot frame 전방 (+x) 방향 속도이므로 world-frame 위치 변화에는 현재 heading rotation이 필요하다.

## 4. Discrete pose integration

Euler integration:

[
x_{k+1}approx x_k+v_kcos	heta_kDelta t
]

[
y_{k+1}approx y_k+v_ksin	heta_kDelta t
]

[
	heta_{k+1}=	heta_k+omega_kDelta t
]

회전 중에는 구간 전체에 시작 heading을 사용하므로 큰 (Delta t)에서 원호를 접선 방향 선분으로 근사하는 오차가 커진다.

Midpoint approximation:

[
	heta_{	ext{mid}}=	heta_k+rac{omega_kDelta t}{2}
]

[
x_{k+1}approx x_k+v_kcos	heta_{	ext{mid}}Delta t
]

[
y_{k+1}approx y_k+v_ksin	heta_{	ext{mid}}Delta t
]

## 5. Encoder increments to body increments

Encoder가 측정한 좌우 wheel rotation increment를 (Deltaphi_L,Deltaphi_R), wheel radius를 (r_L,r_R)라 하면 no-slip 가정 아래:

[
Delta s_L=r_LDeltaphi_L,qquad
Delta s_R=r_RDeltaphi_R
]

Robot 중심 이동거리와 방향각 변화는:

[
Delta s=rac{Delta s_L+Delta s_R}{2}
]

[
Delta	heta=rac{Delta s_R-Delta s_L}{b}
]

이다.

Encoder가 직접 측정하는 것은 차체 운동이 아니라 wheel rotation이다. Wheel-ground contact model이 깨지면 encoder 자체가 정상이어도 body-motion estimate는 틀릴 수 있다.

## 6. Nonholonomic constraint

이상적인 differential-drive robot의 body-frame velocity는:

[
{}^Rmathbf v=
egin{bmatrix}
v_x\0\omega
end{bmatrix}
]

이며 instantaneous lateral velocity (v_y=0)을 가정한다.

이는 옆 위치에 영원히 갈 수 없다는 뜻이 아니라, 현재 순간에 순수 측면 평행이동을 만들 수 없다는 뜻이다. 전진·회전·후진을 조합하면 측면 위치에 도달할 수 있다.

다음 상황에서는 가정이 깨질 수 있다.

- 빙판, 모래, 자갈, 진흙
- 경사면의 측면 slip
- 급회전
- 외력이나 충돌
- wheel lift / loss of contact
- skid-steer의 의도적인 tire scrubbing

## 7. Systematic and situational errors

### Common wheel-radius scale error

좌우 radius를 모두 같은 비율로 크게 설정하면 직진거리 scale을 과대추정한다. 완전한 직진에서는 좌우 차이가 0이므로 yaw는 0으로 유지되지만, 회전량 scale에는 영향이 생긴다.

### Left/right radius mismatch

설정된 (r_R>r_L)이고 encoder angle increment가 같으면:

[
Delta s_R>Delta s_L
]

로 계산되어 실제 직진을 왼쪽 회전으로 오인한다. 반복되면 trajectory가 한 방향으로 휘는 systematic drift가 생긴다.

### Wheelbase error

[
Delta	heta=rac{Delta s_R-Delta s_L}{b}
]

이므로 설정한 (b)가 실제보다 작으면 회전량을 과대추정한다.

### Wheel slip

오른쪽 wheel이 헛돌면 encoder가 (Delta s_R)을 실제보다 크게 보고하여 전진거리와 왼쪽 회전량을 동시에 과대추정할 수 있다.

## 8. Multi-sensor consistency

Wheel encoder, gyro, LiDAR는 같은 입력을 받는 센서가 아니라 서로 다른 물리량을 관측한다.

- encoder: wheel rotation
- gyro: body angular velocity
- LiDAR scan matching: environment geometry가 암시하는 relative pose

Wheel와 gyro의 yaw increment disagreement는 개념적으로:

[
r_	heta=
Delta	heta_{	ext{wheel}}
-
Delta	heta_{	ext{gyro}}
]

라는 residual로 볼 수 있다.

Covariance weighting은 중요하지만, 정상상태 covariance만으로 slip이나 sensor fault를 자동 해결할 수는 없다. Innovation gating, outlier rejection, robust loss, 상황별 covariance 조정 등이 필요할 수 있다.

## 9. Jacobian and covariance propagation

[
x_{k+1}=x_k+vcos	heta_kDelta t
]

[
y_{k+1}=y_k+vsin	heta_kDelta t
]

이므로 heading sensitivity는:

[
rac{partial x_{k+1}}{partial	heta_k}
=-vsin	heta_kDelta t
]

[
rac{partial y_{k+1}}{partial	heta_k}
=vcos	heta_kDelta t
]

이다.

(	heta=0)에서는 yaw uncertainty가 1차적으로 (y) uncertainty로 전달된다.

[
P_{k+1}=F_kP_kF_k^	op+Q_k
]

에서:

- (F_k): 기존 state error를 다음 state로 전달
- (Q_k): 이번 motion increment에서 새로 추가되는 uncertainty

단순화하면 (a=vDelta t)일 때 yaw variance가 추가하는 lateral variance는:

[
a^2operatorname{Var}(	heta)
]

이다.

작은 heading error (delta	heta)로 거리 (L)을 진행할 때 횡오차는:

[
e_y=Lsindelta	hetaapprox Ldelta	heta
]

이다. 따라서 작은 yaw error도 이동거리가 길어지면 큰 lateral drift로 바뀐다.

## 10. Role in a real ground-robot SLAM system

Wheel odometry는:

- high-rate, low-latency local motion estimate
- 외부 조명이나 texture에 덜 의존하는 proprioceptive measurement
- metric distance와 motion constraint
- LiDAR / vision registration의 initial guess
- EKF나 factor-graph estimator의 relative-motion information

을 제공한다.

하지만 global truth가 아니며 누적 drift와 model failure를 피할 수 없다. IMU, LiDAR, vision, GNSS, loop closure 등 독립적인 constraint와 함께 사용해야 한다.

## Final mental model

> Wheel odometry는 encoder의 wheel rotation을 no-slip differential-drive model을 통해 body motion으로 해석한 상대이동 추정이다. 짧은 시간에는 유용하지만, slip과 calibration error가 motion model을 깨뜨리고 작은 yaw error가 장거리 lateral drift로 전파되므로 covariance와 독립적인 sensor constraints가 필요하다.
