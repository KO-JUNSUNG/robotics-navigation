# SO(3), SE(3), and Sensor-Frame Reasoning Memo

> 목적: SE(2)에서 3D rigid-body geometry로 확장한 이유와, 그 geometry가 LiDAR extrinsic 및 IMU measurement 해석에 어떻게 사용되는지를 복원하기 위한 장기 개념 memo.

## Storyline

\`\`\`text
SE(2)는 x, y, yaw만 표현
→ slope와 3D sensor orientation을 표현할 수 없음
→ SO(3) rotation
→ translation을 결합한 SE(3)
→ sensor extrinsic composition
→ lever-arm effect
→ gravity vector와 accelerometer specific force
→ roll/pitch는 gravity로 제약되지만 yaw는 제약되지 않음
→ gyro body-frame increment로 orientation propagation
\`\`\`

## 1. Why SE(2) is insufficient

평지의 두 robot이 같은 \((x,y,\theta)\)를 가져도 하나가 slope 위에 있으면 roll/pitch가 다르다. SE(2)는 이 차이를 표현하지 못한다.

이를 무시하면:

- 3D LiDAR point cloud를 world frame으로 잘못 변환하여 floor/wall이 기울거나 map이 왜곡될 수 있다.
- IMU gravity direction을 잘못 해석할 수 있다.
- 중력 보상 오류가 fictitious linear acceleration과 drift를 만든다.

## 2. SO(3)

3D rotation matrix의 집합은:

\[
SO(3)=
\left\{
R\in\mathbb R^{3\times3}
\mid R^\top R=I,\ \det R=1
\right\}
\]

이다.

- \(R^\top R=I\): 열벡터가 unit length이며 서로 직교한다.
- \(\det R=1\): reflection을 제외하고 handedness를 보존하는 rotation만 허용한다.
- 따라서 \(R^{-1}=R^\top\).

Reflection은 길이와 각도는 보존하지만 오른손 좌표계를 왼손 좌표계로 바꾸며, rigid body를 연속적으로 회전시켜 얻을 수 없다.

## 3. Columns of a rotation matrix

\(R_{WR}\)가 robot coordinates를 world coordinates로 옮긴다면:

\[
{}^Wp=R_{WR}{}^Rp
\]

이고:

\[
R_{WR}
=
\begin{bmatrix}
\vert&\vert&\vert\\
{}^We_x^R&{}^We_y^R&{}^We_z^R\\
\vert&\vert&\vert
\end{bmatrix}
\]

이다.

즉 각 열은 robot의 \(x,y,z\)축을 world frame으로 표현한 것이다. Basis vector를 곱하면 해당 열이 선택된다.

예를 들어 \(+90^\circ\) yaw에서는:

\[
R_z(90^\circ)=
\begin{bmatrix}
0&-1&0\\
1&0&0\\
0&0&1
\end{bmatrix}
\]

이므로 robot \(+x\)는 world \(+y\), robot \(+y\)는 world \(-x\), robot \(+z\)는 world \(+z\)를 가리킨다.

## 4. 3D rotations are noncommutative

일반적으로:

\[
R_yR_z\neq R_zR_y
\]

이다. 3D에서는 회전 순서와 회전축이 world-fixed인지 body-moving인지가 결과를 바꾼다.

행렬은 오른쪽부터 적용된다.

\[
R_yR_zv
\]

는 먼저 \(R_z\), 그다음 \(R_y\)를 적용한다.

Euler-angle 순서 규칙을 고립된 암기 대상으로 삼기보다, 실제 구현에서 frame label과 rotation convention을 확인한다.

## 5. Frame-labeled rotation composition

현재 orientation이 \(R_{WR_1}\), body-relative rotation이 \(R_{R_1R_2}\)라면:

\[
R_{WR_2}=R_{WR_1}R_{R_1R_2}
\]

이다.

벡터 변환으로 보면:

\[
{}^{R_1}p=R_{R_1R_2}{}^{R_2}p
\]

\[
{}^Wp=R_{WR_1}{}^{R_1}p
\]

이므로:

\[
{}^Wp
=
R_{WR_1}R_{R_1R_2}{}^{R_2}p
\]

이다. Frame path는 오른쪽부터 \(R_2\rightarrow R_1\rightarrow W\)이다.

## 6. SE(3)

3D pose는 rotation과 translation을 결합한다.

\[
{}^Wp=R_{WR}{}^Rp+{}^Wt_R
\]

- \(R_{WR}\): robot vector를 world 방향으로 회전
- \({}^Wt_R\): robot 원점의 world-frame 위치
- \({}^Rp\), \({}^Wp\): 같은 point의 robot/world coordinates

Homogeneous transform은:

\[
T_{WR}
=
\begin{bmatrix}
R_{WR}&{}^Wt_R\\
0&1
\end{bmatrix}
\in SE(3)
\]

이다.

Composition은 SE(2)와 동일하게 frame path를 따른다.

\[
T_{AB}T_{BC}=T_{AC}
\]

## 7. SE(3) inverse

\[
{}^Wp=R_{WR}{}^Rp+{}^Wt_R
\]

를 robot coordinates에 대해 풀면:

\[
{}^Rp
=
R_{WR}^\top
\left(
{}^Wp-{}^Wt_R
\right)
\]

따라서:

\[
T_{RW}
=
T_{WR}^{-1}
=
\begin{bmatrix}
R_{WR}^\top&
-R_{WR}^\top{}^Wt_R\\
0&1
\end{bmatrix}
\]

이다.

Inverse translation은 단순한 \(-{}^Wt_R\)가 아니다. Translation을 inverse destination frame으로 회전해야 한다.

## 8. Sensor extrinsic

LiDAR frame을 \(L\), robot frame을 \(R\)이라 하면:

\[
T_{RL}
=
\begin{bmatrix}
R_{RL}&{}^Rt_L\\
0&1
\end{bmatrix}
\]

이다.

- \(R_{RL}\): LiDAR vector를 robot frame으로 회전
- \({}^Rt_L\): LiDAR 원점의 위치를 robot frame으로 표현
- \(T_{RL}\): LiDAR coordinates를 robot coordinates로 변환

Point transformation:

\[
{}^Rp
=
R_{RL}{}^Lp+{}^Rt_L
\]

Robot world pose와 합성하면:

\[
T_{WL}=T_{WR}T_{RL}
\]

이다.

반대로 scan matching이 \(T_{WL}\)을 추정하고 robot pose가 필요하면:

\[
T_{WR}=T_{WL}T_{LR}
\]

이다.

## 9. Lever-arm effect

LiDAR가 robot 회전중심에서 떨어져 장착되면 robot이 제자리 회전해도 LiDAR 원점은 원호를 그리며 translation한다.

예를 들어 LiDAR가 전방 1 m에 있고 robot이 \(180^\circ\) 제자리 회전하면:

- robot 원점 translation: 0
- LiDAR 시작 위치: \((1,0)\)
- LiDAR 종료 위치: \((-1,0)\)
- LiDAR 원점의 직선 displacement: \((-2,0)\)

Extrinsic을 무시하고 \(T_{WL}\)을 \(T_{WR}\)로 사용하면 lever-arm 회전으로 생긴 sensor translation을 robot translation으로 오인한다.

## 10. Point versus vector

Point에는 rotation과 translation을 모두 적용한다.

\[
{}^Wp=R_{WR}{}^Rp+{}^Wt_R
\]

Direction, angular velocity, gravity 같은 vector에는 translation을 적용하지 않는다.

\[
{}^Wv=R_{WR}{}^Rv
\]

## 11. Gravity and accelerometer specific force

World gravity vector를:

\[
{}^Wg=
\begin{bmatrix}
0\\0\\-g
\end{bmatrix}
\]

라고 하면 IMU frame의 gravity는:

\[
{}^Ig=R_{IW}{}^Wg
\]

이다.

Ideal accelerometer는 world kinematic acceleration 자체가 아니라 specific force를 측정한다.

\[
{}^If
=
R_{IW}
\left(
{}^Wa_I-{}^Wg
\right)
\]

정지 상태에서는 \({}^Wa_I=0\)이므로, 축이 정렬되어 있다면:

\[
{}^If=
\begin{bmatrix}
0\\0\\+g
\end{bmatrix}
\]

이다.

자유낙하에서는 \({}^Wa_I={}^Wg\)이므로:

\[
{}^If=0
\]

이다. 중력이 사라진 것이 아니라 IMU 전체가 중력과 함께 가속되어 supporting specific force가 없는 것이다.

## 12. Gravity observability

Roll 또는 pitch가 변하면 gravity vector의 IMU-axis projection이 달라진다. 따라서 정적이거나 선형가속도가 충분히 작을 때 gravity는 roll/pitch 정보를 제공한다.

하지만 yaw는 gravity vector 자체를 축으로 하는 회전이다.

\[
R_z(\psi)
\begin{bmatrix}
0\\0\\-g
\end{bmatrix}
=
\begin{bmatrix}
0\\0\\-g
\end{bmatrix}
\]

따라서 accelerometer만으로 yaw를 알 수 없다. Yaw correction에는 magnetometer, wheel constraint, LiDAR/vision, GNSS heading 등의 추가 정보가 필요하다.

## 13. Gyro body-frame propagation

Gyro measurement는 일반적으로 IMU frame으로 표현된다.

\[
\omega_m=\omega_{\text{true}}+b_g+n_g
\]

시점 \(k\)에서 gyro를 적분해 얻은 relative rotation을 \(R_{I_kI_{k+1}}\)라 하면:

\[
R_{WI_{k+1}}
=
R_{WI_k}R_{I_kI_{k+1}}
\]

이다.

Frame path는:

\[
I_{k+1}\rightarrow I_k\rightarrow W
\]

이다.

“Body increment는 오른쪽”이라는 규칙은 현재 rotation definition과 relative-rotation convention 아래에서 성립한다. 다른 convention에서는 식이 달라질 수 있으므로 frame label을 우선한다.

## Final mental model

> SO(3)는 orientation을, SE(3)는 orientation과 position을 함께 표현한다. Frame-labeled composition을 사용하면 robot, LiDAR, IMU 사이의 변환과 inverse를 일관되게 구성할 수 있다. Sensor extrinsic을 무시하면 lever-arm motion을 robot motion으로 오인할 수 있고, IMU에서는 gravity와 specific force를 구분해야 한다. Gravity는 roll/pitch를 제약하지만 yaw에는 불변이므로 gyro bias에 의한 yaw drift를 막으려면 추가 sensor constraint가 필요하다.
