# Experiments Skeleton

## Goal

Show whether minimal warm-start / transfer is enough to turn the dual-bank DeePC scaffold into a useful fault-aware prototype.

## Main matrix

- trajectories: `step`, `figure8`
- scenarios: `mild_single_rotor_drop`, `severe_single_rotor_drop`
- onset: `20.0s`
- controllers:
  - `deepc_nominal_bank`
  - `deepc_degraded_bank`
  - `deepc_degraded_transfer`
  - `mpc`
- seeds:
  - fixed set of `10` seeds for the strengthened paper matrix, with `41-45` reused and `46-50` newly added

## Metrics

- position RMSE
- final position error norm
- yaw RMSE
- success rate
- mean and std across seeds
- 95 percent CI or error bars
- transfer delta relative to the old banks

## What each result answers

- Main table: whether transfer changes the average performance profile.
- Delta table: where transfer helps and where it is still incomplete.
- Error bars or CI: whether the gain is seed-robust or seed-specific.
- Advantage case plot: where transfer clearly helps.
- Boundary case plot: where the gain is partial or uneven.

## Figures

- Main text figure:
  - representative advantage case
  - main table
- Appendix figure:
  - boundary case
  - transfer delta table
  - mean old-bank vs transfer summary plot

## Claim boundary

This experiment supports a conservative claim:

- minimal transfer is necessary for a fault-aware DeePC prototype with average gains under single-rotor degradation

It does not support:

- universal superiority over all cases
- superiority over MPC
- final controller status
