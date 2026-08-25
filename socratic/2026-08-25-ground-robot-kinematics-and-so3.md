# Socratic Session — 2026-08-25 Ground-Robot Kinematics, Wheel Odometry, and SO(3) Start

## Purpose

Establish the practical differential-drive and wheel-odometry foundation, connect it to uncertainty and sensor fusion, then begin the prerequisite transition from SE(2) to SO(3).

## Reliable reasoning demonstrated

The learner can now reason through:

- (v=(v_L+v_R)/2) and (omega=(v_R-v_L)/b)
- turning direction and instantaneous turning radius
- encoder angle increments to wheel displacement
- center displacement and yaw increment
- Euler versus midpoint integration intuition
- no-slip and nonholonomic assumptions
- wheel slip, radius mismatch, and wheelbase calibration errors
- why encoder rotation is not identical to body motion
- why wheel odometry remains useful as high-rate local information
- disagreement among wheel odometry, gyro, and LiDAR
- small heading error becoming lateral position error
- motion-model Jacobian transmitting yaw uncertainty into lateral covariance

## Important corrections

### Angular velocity

Initial uncertainty: angular velocity might be related to differentiating acceleration.

Correction: yaw angular velocity is (omega=dot{	heta}). Angular acceleration is (dot{omega}); jerk is the derivative of linear acceleration.

### Slip affects both translation and rotation

Initial answer: right-wheel slip would overestimate forward motion but might leave rotation unchanged.

Correction: because (Delta	heta=(Delta s_R-Delta s_L)/b), over-reporting right-wheel travel can also overestimate left rotation.

### Encoder interpretation

Refinement: encoder measures wheel rotation, not body displacement. A lifted robot with spinning wheels is a strong counterexample to treating wheel odometry as ground truth.

### Gyro terminology

Initial wording referred to measured angular momentum.

Correction: a gyro measures angular velocity, with noise, bias, scale, vibration, saturation, and timing effects depending on the sensor/system.

### Covariance propagation

Initial reasoning described Jacobian terms as repeatedly being squared.

Correction: (F P F^	op) maps existing state uncertainty between components. Near (	heta=0), the coupling (partial y_{k+1}/partial	heta_k=vDelta t) transfers yaw variance into lateral position variance.

## SO(3) progress

Introductory working understanding:

- SE(2) cannot represent slope-induced roll/pitch needed by IMU and 3D LiDAR.
- (R^	op R=I), (det R=1), and reflection is excluded.
- Columns of (R_{WR}) are robot axes expressed in world coordinates.
- (R^{-1}=R^	op).
- Yaw and pitch matrices can be applied to basis vectors.
- 3D rotations are noncommutative.
- Frame-labeled composition (R_{WR_2}=R_{WR_1}R_{R_1R_2}) is understood.

## Unresolved / defer deliberately

Body-fixed versus world-fixed rotation composition did not yet become intuitive when presented abstractly. Do not mark it completed or drill matrix-order rules in isolation.

Revisit it when gyro integration creates a concrete need:

```text
gyro angular increment expressed in IMU/body frame
→ current orientation
→ frame-labeled rotation composition
→ right/left perturbation convention
```

## Next

Continue SO(3) at the physical frame-transformation level, then extend to SE(3). Avoid an Euler-angle convention detour. Use IMU / LiDAR extrinsics as the practical application before EKF / ESKF.
