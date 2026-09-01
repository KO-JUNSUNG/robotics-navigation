# LiDAR Geometry and Scan-Matching Foundations Memo

> 목적: 2D ground-robot LiDAR scan의 frame, correspondence, ICP, geometry-induced degeneracy, deskew, extrinsic 변환을 하나의 실전 mental model로 복원한다. ICP 선형화와 구현은 다음 학습 범위다.

## Storyline

```text
서로 다른 시각의 scan은 서로 다른 LiDAR frame에 표현
→ 공통 frame으로 옮길 상대 pose가 필요
→ correspondence와 transform을 번갈아 추정하는 ICP
→ initial guess, outlier, 반복 구조와 degeneracy 문제
→ point-to-line geometry가 관측 가능한 motion 방향을 결정
→ scan 내부 timestamp 차이는 deskew로 보정
→ LiDAR motion을 extrinsic으로 robot motion으로 변환
→ pose뿐 아니라 covariance도 lever arm에 맞게 변환
```

## 1. Scan points live in different frames

두 시각의 같은 물리적 점도 좌표는 서로 다르다.

\[
{}^{L_k}p
=
T_{L_kL_{k+1}}{}^{L_{k+1}}p
\]

World pose를 알 필요는 없지만 두 scan을 비교하려면 같은 frame으로 변환해야 한다. 같은 beam index는 같은 LiDAR-relative angle일 뿐 같은 물리적 표면점을 보장하지 않는다.

## 2. ICP and correspondence

Point-to-point objective:

\[
\min_{R,t}
\sum_{(i,j)\in\mathcal C}
\left\|{}^{L_k}p_i-\left(R{}^{L_{k+1}}p_j+t\right)\right\|^2
\]

ICP는 현재 transform으로 correspondence를 찾고, 그 correspondence의 residual을 줄이는 transform을 다시 계산한다. Correspondence와 transform이 서로 의존하므로 local optimization이며, wheel/IMU initial guess가 올바른 basin에 들어가는 데 중요하다.

## 3. Point-to-line and geometry

평면/선 구조에서는 point-to-line residual이 자연스럽다.

\[
r_i=n_i^\top\left(Rp_i+t-q_i\right)
\]

법선 \(n_i\)은 벽에 수직인 방향이며 residual은 motion의 법선 성분을 측정한다. 복도 길이를 \(x\), 폭을 \(y\)로 두면 옆 벽의 법선 \([0,1]^\top\)은 횡이동을 제약하지만 종이동은 거의 제약하지 못한다. 끝 벽의 법선 \([1,0]^\top\)이 보이면 종이동도 제약된다.

환경 geometry는 Jacobian과 Hessian의 강한/약한 방향을 결정한다. 평행 벽만 있는 복도는 종방향 cost가 연속적으로 평평한 degeneracy를 만든다. 반복 기둥은 기둥 간격마다 별도의 local minimum을 만드는 perceptual aliasing 문제다. 작은 최종 residual만으로 올바른 pose를 보장할 수 없다.

## 4. Scan deskew

회전형 LiDAR의 한 scan은 일정 시간이 걸리므로 각 점의 source frame이 다르다.

\[
p_1^{L(t_1)},\ldots,p_N^{L(t_N)}
\]

모든 점에 같은 transform을 적용하면 회전/이동 중 벽이 휘거나 기울어지는 motion distortion이 남는다. IMU/wheel trajectory를 point timestamp로 보간하여 각 점을 공통 reference frame으로 옮긴다.

\[
{}^{L_{\text{ref}}}p_i
=
T_{L_{\text{ref}}L(t_i)}{}^{L(t_i)}p_i
\]

## 5. LiDAR motion to robot motion

고정 extrinsic \(T_{RL}:L\rightarrow R\)에 대해:

\[
T_{R_kR_{k+1}}
=
T_{RL}T_{L_kL_{k+1}}T_{LR}
\]

frame path는:

\[
R_{k+1}\rightarrow L_{k+1}\rightarrow L_k\rightarrow R_k
\]

이다. LiDAR가 robot 회전 중심에서 떨어져 있으면 제자리 yaw 중 LiDAR 원점은 원호를 움직인다. LiDAR translation을 robot-center translation으로 그대로 사용하면 false translation이 생긴다.

Lever arm은 covariance에도 영향을 준다. 작은 각도에서 거리 \(r\)인 센서의 yaw uncertainty \(\sigma_\theta\)는 대략:

\[
\sigma_p\approx r\sigma_\theta
\]

의 위치 uncertainty를 만든다. 따라서 LiDAR-frame covariance를 body-frame covariance로 숫자만 복사해서는 안 된다.

## 6. Practical front-end cycle

```text
point timestamp 확인
→ IMU/wheel trajectory로 deskew
→ motion initial guess
→ correspondence 탐색
→ point-to-point / point-to-line optimization
→ convergence, overlap, residual, degeneracy, physical consistency 검사
→ relative pose와 방향별 covariance 출력
→ ESKF 또는 factor graph에 전달
```

## Current boundary

신뢰 가능한 범위:

- scan frame 간 relative transform 의미와 부호
- beam index가 물리적 correspondence가 아닌 이유
- ICP의 alternating structure와 initial-guess dependence
- point-to-line normal과 corridor degeneracy
- continuous degeneracy와 repeated-structure local minima 구분
- per-point timestamp와 deskew 필요성
- LiDAR-relative pose를 robot-relative pose로 바꾸는 extrinsic composition
- lever-arm rotation이 translation과 covariance에 미치는 영향

다음 시작점:

- \(r\sigma_\theta\) 수치 확인
- point-to-line ICP의 작은-pose linearization과 Jacobian 직관
- correspondence rejection, robust loss, convergence/quality metrics
- LiDAR odometry measurement를 ESKF/factor graph에 넣는 frame과 covariance 설계
