# Paper Outline

## Working Title

`Minimal Warm-Start / Transfer for Fault-Aware DeePC under Single-Rotor Degradation`

## 1. Introduction

- Problem: DeePC dual-bank control under actuator degradation can freeze into the same data state when the degraded bank does not keep adapting after fault onset.
- Gap: a nominal/degraded scaffold alone does not yet produce a defensible fault-aware method claim.
- Our response: add a minimal warm-start / transfer mechanism so the degraded bank continues to adapt after fault onset.
- Claim boundary: this is a lightweight quadrotor simulation prototype with bounded gains, not a final controller and not a stronger-than-MPC solution.

## 2. Related Problem Setup and Motivation

- Why single-rotor efficiency drop is a good controlled fault setting.
- Why the old dual-bank scaffold is the right baseline to diagnose.
- Why the comparison set must include `nominal_bank`, `degraded_bank`, transfer, and `mpc`.

## 3. Method

- The nominal/degraded scaffold.
- Why the scaffold is insufficient without transfer.
- The minimal warm-start / transfer mechanism.
- What is and is not changed relative to the old scaffold.

## 4. Experimental Protocol

- Frozen benchmark matrix.
- Seeds and aggregation.
- Metrics, error bars, and confidence intervals.
- Case plots and delta tables.

## 5. Results

- Main table: aggregated controller performance.
- Delta table: transfer relative to old banks.
- Representative advantage case.
- Representative boundary case.
- Mean improvements and remaining gaps.

## 6. Discussion

- Why transfer is necessary for the current fault-aware DeePC claim.
- Why the result is still not a final controller win.
- Why MPC remains a reference, not the target claim.

## 7. Conclusion

- Minimal transfer changes the dual-bank scaffold into a usable fault-aware prototype.
- The result is a conservative method contribution with clear but bounded gains.
