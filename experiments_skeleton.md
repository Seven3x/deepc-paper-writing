# Experiments Skeleton

## Experiment question

Test whether minimal warm-start / transfer is enough to make the degraded bank practically useful in the fixed single-rotor degradation setup.

## Fixed main matrix (paper-locked)

- trajectories: `step`, `figure8`
- scenarios: `mild_single_rotor_drop`, `severe_single_rotor_drop`
- fault onset: `20.0s`
- controllers: `deepc_nominal_bank`, `deepc_degraded_bank`, `deepc_degraded_transfer`, `mpc`
- seeds: `41-50` (10 seeds total)
- total runs: `2 x 2 x 4 x 10 = 160`

## Metrics and aggregation

- primary metrics: position RMSE, final position error norm, yaw RMSE
- reliability metric: success rate
- per-case aggregation: mean and std across the 10 seeds
- uncertainty display: 95% CI / error bars for seed-level mean estimates
- transfer effect view: `deepc_degraded_transfer - old_bank` deltas, where old_bank is nominal/degraded baseline (identical in this matrix)

## Result artifact roles

- main table (`track2_main_results_table.*`): absolute performance profile of all four controllers on all four case combinations
- delta table (`track2_degraded_transfer_deltas_table.*`): where transfer improves vs old dual-bank and where it regresses
- advantage case figure (`track2_case_step_severe.*`): representative combination where transfer improves across core DeePC metrics vs old banks
- boundary case figure (`track2_case_figure8_mild.*`): representative combination with mixed outcome and explicit remaining weakness
- CI bars (`track2_main_matrix_ci_bars.*`): seed-robustness check for average gains and boundary variability

## Placement plan

- main text: main table, CI bars, advantage case
- appendix: delta table, boundary case, seed-level summary, delta heatmap, average old-vs-transfer overview

## Claim boundary carried by this section

- supported: transfer introduces stable average improvement over old dual-bank DeePC in this fixed setting
- not supported: all-case/all-metric dominance, superiority to MPC, mature final fault-tolerant controller claim
