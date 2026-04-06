# Results Skeleton

## 1. Main result (absolute profile)

- Use the main results table to show all controllers on all four case combinations.
- State the structural observation first: `deepc_nominal_bank` and `deepc_degraded_bank` are numerically identical in this matrix.
- Then state transfer behavior: `deepc_degraded_transfer` changes this frozen pattern and produces non-trivial separation from old banks.

## 2. Improvement over old dual-bank baseline

- Report per-case deltas of transfer minus old bank for position RMSE, final position error, yaw RMSE.
- Report cross-case average deltas:
- position RMSE: `-10.21`
- final position error: `-99.12`
- yaw RMSE: `-649.91`
- Interpret conservatively: average effect is positive, indicating fault-aware gain relative to the old scaffold.

## 3. Boundary and failure-mode presentation

- Explicitly keep mixed cases visible:
- `step + mild`: final position regresses (`+83.80`) despite position/yaw gains.
- `figure8 + mild`: position (`+3.96`) and yaw (`+46.78`) regress, while final position improves.
- Use boundary case figure and CI bars to show that the method is improved on average but still uneven.

## 4. MPC as reference only

- Keep an explicit sentence that MPC remains stronger in key position/final-position metrics in this benchmark.
- Do not structure the section as "transfer vs MPC win/loss table"; use MPC only to anchor maturity level.

## 5. Section closing sentence template

- "These results support a conservative method claim: minimal transfer makes the dual-bank DeePC scaffold fault-aware with stable average gains, while leaving clear boundary cases and a performance gap to MPC."
