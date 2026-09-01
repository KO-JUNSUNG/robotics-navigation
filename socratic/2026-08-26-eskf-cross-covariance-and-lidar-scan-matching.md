# Socratic Session — 2026-08-26 to 2026-09-01 ESKF Cross-Covariance and LiDAR Scan Matching

## Reliable reasoning demonstrated

The learner can now reason through:

- excessive estimated gyro bias causing under-integrated yaw
- the sign of yaw-error / gyro-bias-error cross-covariance under an explicit error convention
- \(FPF^\top\) creating cross-covariance from initially diagonal uncertainty
- a yaw-only \(H=[1\ 0]\) producing a non-zero bias Kalman-gain row through \(PH^\top\)
- cross-covariance as a correction pathway rather than a source of observability
- additive gyro bias versus multiplicative wheel scale patterns
- excitation using stationary, different-rate, and opposite-direction rotations
- wheel-slip / LiDAR-failure consistency risks and adaptive down-weighting versus rejection
- asynchronous IMU propagation and wheel/LiDAR update ordering
- measurement covariance update, duplicate-measurement overconfidence, and indirect bias corruption
- scan points from different timestamps living in different LiDAR frames
- ICP correspondence / transform coupling and initial-guess dependence
- point-to-line normals and corridor-direction degeneracy
- continuous degeneracy versus repeated-structure local minima
- scan deskew as per-point transformation to a common reference time
- LiDAR-to-robot relative-transform composition using fixed extrinsics
- LiDAR lever-arm motion during robot-center yaw rotation

## Corrections and refinements

### Cross-covariance versus observability

Initial answer suggested cross-covariance might resolve gyro-only true-rotation/bias ambiguity. Correction: it only transmits correction; an independent reference and residual are required to distinguish state hypotheses.

### Bias versus scale pattern

Initial answer classified a constant signed gyro-wheel offset as multiplicative scale error. Correction: constant signed offset is additive bias; scale error changes magnitude with motion and flips signed error with rotation direction.

### Measurement failure and covariance

The learner correctly anticipated a failure loop from trusting slipped wheel odometry. Refined distinction: actual error can grow while reported covariance shrinks, producing inconsistency. Repeated bad measurements may accumulate even when down-weighted; hard rejection protects the state but discards potentially useful information.

### ICP failure modes

Initial reasoning grouped repeating pillars with featureless-corridor degeneracy. Correction: parallel walls give a continuous weak direction, whereas repeated pillars give multiple separated local minima through wrong but plausible correspondences.

### Deskew source frames

The learner understood that a single rigid transform cannot correct a rotating scan. Refined wording: each point has a different source frame \(L(t_i)\), and point-specific transforms map them into one common target/reference frame.

## Current frontier

The session stopped after establishing that a LiDAR lever arm converts yaw uncertainty into position uncertainty approximately as:

\[
\sigma_p\approx r\sigma_\theta
\]

The exact next retrieval question is to compute this for \(r=0.2\,\mathrm m\), \(\sigma_\theta=0.1\,\mathrm{rad}\), then proceed to point-to-line ICP linearization and robust correspondence handling.
