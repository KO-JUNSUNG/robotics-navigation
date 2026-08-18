# Socratic Session — 2026-08-18 Initial Diagnostic

## Purpose

Initial knowledge assessment before beginning the structured SLAM curriculum.

## Learner Preference

Use Socratic questioning. Preserve original answers, then provide precise correction and identify the next knowledge gap.

## Questions Covered

The session covered:

- A1: wheel odometry / dead reckoning intuition
- A2: IMU integration and error growth
- A3: camera-only localization and monocular limitations
- A4: Kalman filtering and MAP
- A5: covariance and sensor trust
- A6: why localization and mapping are coupled
- A7: initial SLAM pipeline mental model
- A8: 2D frame transformation
- A9: inverse rotation and transform direction
- A10: motion and measurement models
- A11: Bayesian posterior structure
- A12: estimate + uncertainty
- A13: body-frame motion and pose composition
- A14: relative pose composition

The full assessment, including original answers and detailed evaluation, is in `assessments/initial-diagnostic.md`.

## Key Discoveries

### Strong

The learner already has good intuition for:

- uncertainty
- Bayesian reasoning
- covariance as sensor confidence
- state estimation
- why inertial integration drifts
- localization/mapping coupling

### Major Gaps

The most important missing foundation is robotics geometry:

- coordinate-frame notation
- transform direction
- SO(3)/SE(3)
- pose composition
- relative pose

This should be learned before going deeply into EKF/ESKF or SLAM algorithms.

## Important Corrections Made

1. Monocular VO is possible; metric scale is the central classical ambiguity.
2. IMU is a measurement source that is commonly used for state propagation.
3. Kalman filtering is not synonymous with MAP estimation in general.
4. Rotation inverse equals transpose due to orthonormality.
5. Odometry error does not universally grow as t^2.
6. A robot-frame displacement must be rotated into the world frame before updating world position.

## Immediate Next Step

Continue with the pose-landmark relationship:

`^R p = f(T_WR, ^W p)`

Then formulate SLAM as joint estimation of unknown poses and map landmarks under sensor constraints.
