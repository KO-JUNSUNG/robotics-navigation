# Socratic Session — 2026-08-25 Ground-Robot Kinematics, SO(3)/SE(3), and ESKF Foundations

## Purpose

Establish the practical differential-drive / wheel-odometry foundation, extend SE(2) reasoning to SO(3) and SE(3), apply frame transforms to LiDAR and IMU, then build an introductory ESKF mental model.

## Reliable reasoning demonstrated

The learner can now reason through:

- \(v=(v_L+v_R)/2\) and \(\omega=(v_R-v_L)/b\)
- turning direction and instantaneous turning radius
- encoder angle increments to wheel displacement
- center displacement and yaw increment
- Euler versus midpoint integration intuition
- no-slip and nonholonomic assumptions
- wheel slip, radius mismatch, and wheelbase calibration errors
- why encoder rotation is not identical to body motion
- disagreement among wheel odometry, gyro, and LiDAR
- heading error becoming lateral position error
- motion-model Jacobian transmitting yaw uncertainty into lateral covariance
- SO(3) determinant, orthonormality, axis columns, inverse, and noncommutativity
- SE(3) transform composition and inverse
- LiDAR extrinsic composition and lever-arm failure
- accelerometer specific force in stationary and free-fall cases
- gravity-based roll/pitch observability and yaw unobservability
- gyro bias accumulation and body-frame orientation increment
- ESKF nominal/error state, injection/reset, covariance, and Kalman-gain intuition

## Important corrections

### Angular velocity

Initial uncertainty: angular velocity might be related to differentiating acceleration.

Correction: yaw angular velocity is \(\omega=\dot\theta\). Angular acceleration is \(\dot\omega\); jerk is the derivative of linear acceleration.

### Slip affects both translation and rotation

Initial answer: right-wheel slip would overestimate forward motion but might leave rotation unchanged.

Correction:

\[
\Delta\theta=\frac{\Delta s_R-\Delta s_L}{b}
\]

so over-reporting right-wheel travel can also overestimate left rotation.

### Encoder interpretation

Encoder measures wheel rotation, not body displacement. A lifted robot with spinning wheels is a strong counterexample to treating wheel odometry as ground truth.

### Gyro terminology

Initial wording referred to measured angular momentum.

Correction: a gyro measures angular velocity, with noise, bias, scale, vibration, saturation, and timing effects depending on the sensor/system.

### Covariance propagation

Initial reasoning described Jacobian terms as repeatedly being squared.

Correction: \(FPF^\top\) maps existing state uncertainty between components. Near \(\theta=0\), the coupling:

\[
\frac{\partial y_{k+1}}{\partial\theta_k}=v\Delta t
\]

transfers yaw variance into lateral position variance.

### Body-fixed versus world-fixed rotation

Abstract matrix-order explanation was not initially useful. The concept became reliable only when connected to gyro propagation.

\[
R_{WI_{k+1}}
=
R_{WI_k}R_{I_kI_{k+1}}
\]

The reason is the concrete frame path:

\[
I_{k+1}\rightarrow I_k\rightarrow W
\]

Do not reteach this as an isolated right/left multiplication rule. Start from the sensor frame and transform labels.

### Free fall

Initial explanation suggested gravity was canceled in free fall.

Correction: gravity remains and causes \({}^Wa_I={}^Wg\). Accelerometer specific force is zero because the IMU falls with gravity and has no supporting contact force.

### Error-state reset

Initial answer correctly sensed that an old correction should not remain, but the precise reason is that injection changes the nominal reference. Leaving the same error mean would apply the correction twice.

Resetting \(\delta\hat x\) to zero does not reset \(P\) to zero.

### Kalman residual versus Jacobian

Initial reasoning connected a large residual with a large measurement Jacobian \(H\).

Correction:

- \(r=z-h(x)\) is measurement disagreement.
- \(H=\partial h/\partial x\) is sensitivity.
- A large residual does not itself make \(H\) large.

If \(P\) is unrealistically small, \(K\) becomes small and the estimator can ignore valid LiDAR correction despite a large residual.

## SO(3) / SE(3) progress

Working understanding:

- SE(2) cannot express slope-induced roll/pitch.
- \(R^\top R=I\), \(\det R=1\), and reflection is excluded.
- Columns of \(R_{WR}\) are robot axes expressed in world coordinates.
- \(R^{-1}=R^\top\).
- 3D rotations are noncommutative.
- Frame-labeled composition is reliable.
- SE(3) point transform, composition, and inverse are reliable.
- \(T_{WL}=T_{WR}T_{RL}\) and \(T_{WR}=T_{WL}T_{LR}\).
- Ignoring a LiDAR lever arm turns sensor-origin translation into false robot translation.

Detailed exponential-map computation was explicitly skipped because it was not useful to the learner at this stage. Revisit only for implementation.

## IMU progress

Working understanding:

- Accelerometer measures specific force, not raw world kinematic acceleration.
- Stationary aligned IMU measures approximately \(+g\) upward.
- Free-fall ideal measurement is zero.
- Gravity constrains roll/pitch but is invariant to yaw.
- Gyro-only constant output cannot separate true constant rotation from constant bias.
- Bias changes with temperature, time, vibration, and operating conditions.
- External wheel / LiDAR constraints are needed for yaw and bias correction.

## ESKF progress

Working conceptual understanding:

- nominal state \(x=(p,v,R,b_a,b_g)\)
- small error state \(\delta x=(\delta p,\delta v,\delta\theta,\delta b_a,\delta b_g)\)
- direct matrix addition can break \(SO(3)\)
- rotation correction is injected through a valid small rotation
- error mean resets after injection to avoid double counting
- covariance remains because uncertainty remains
- \(P\) too small produces overconfidence and weak measurement correction
- measurements can indirectly correct correlated states

## Unresolved / next retrieval point

The session ended just before explaining the mechanism:

\`\`\`text
gyro-bias error
→ yaw propagation error
→ yaw/bias cross-covariance
→ wheel or LiDAR yaw residual
→ indirect gyro-bias correction
\`\`\`

Next session should begin with a small conceptual or numeric cross-covariance example, then continue to ESKF observability and consistency. Do not restart SO(3), SE(3), specific force, or basic Kalman-gain material unless retrieval shows a real gap.
