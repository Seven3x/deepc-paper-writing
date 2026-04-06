# Experiment Agent Status

最后更新时间：2026-04-01 18:50

## 结论

- 本轮主实验按冻结矩阵完成了 `5` 个固定随机种子的聚合，而不是 `10` 个。
- 这不是偷工减料：当前 `5-seed` 矩阵已经是 `80` 个仿真运行（`2 trajectories x 2 scenarios x 4 variants x 5 seeds`），并且结果趋势已经非常明确，没有出现 mainline 复活信号。
- 这组聚合是在 fairness 修复后、统一 observation sample 的前提下做的，因此现在可以用于方法比较。
- 聚合结果不支持 milestone 3。
- 没有任何候选在两个核心坏场景上稳定优于 `uniform regularized DeePC`。
- `xyz-only DeePC` 在所有聚合结果里都极弱，因此“打赢 `xyz-only`”本身不构成主方法成立证据。

## Experiment Matrix Actually Run

- trajectories: `step`, `figure8`
- scenarios: `yaw_drift`, `anisotropic_noise`
- seeds: `7, 42, 123, 314, 2026`
- controllers / variants:
  - `uniform`
  - `xyz_only`
  - `measurement_noise`
  - `robust_residual_stats`

结果文件：

- [/home/roxy/deepc-paper/paper/docs/status/aggregated_results.csv](/home/roxy/deepc-paper/paper/docs/status/aggregated_results.csv)
- [/home/roxy/deepc-paper/paper/docs/status/aggregated_results.json](/home/roxy/deepc-paper/paper/docs/status/aggregated_results.json)

## Important Reconciliation Note

- 这里有一个需要明确写出来的偏差：
  - method agent 推荐主实验保留 `measurement_noise` 和 `residual_variance`
  - 但实际 multi-seed runner 跑的是 `measurement_noise` 和 `robust_residual_stats`
- 这意味着：本轮 multi-seed 聚合**没有**完整验证 `residual_variance`。
- 但这不改变当前 go/no-go 判断，因为：
  - method smoke 已经显示 `residual_variance` 没有超过 `measurement_noise`
  - multi-seed 聚合里 `measurement_noise` 本身也没有形成对 `uniform` 的稳定优势
  - 所以 measurement-aware mainline 仍然缺少正证据

## Per-Controller Summary

`uniform`

- 仍然是当前最强、最稳定的参考线。
- `yaw_drift + step` 与 `measurement_noise` 完全同分：
  - `rmse_position_mean = 1.1765`
  - `rmse_yaw_mean = 0.8158`
- `yaw_drift + figure8` 也与 `measurement_noise` 完全同分：
  - `rmse_position_mean = 2.0777`
  - `rmse_yaw_mean = 0.5735`
- `anisotropic_noise` 下虽然性能很差，但依然没有被新候选稳定压过。

`measurement_noise`

- 没有形成稳定优势。
- 在两个 `yaw_drift` 场景里与 `uniform` 完全一样。
- 在 `step + anisotropic_noise` 上位置项略差于 `uniform`，yaw 也更差：
  - `rmse_position_mean = 2.4752` vs `2.4739`
  - `rmse_yaw_mean = 2.8444` vs `1.7407`
- 在 `figure8 + anisotropic_noise` 上也没有超过 `uniform`。

`robust_residual_stats`

- 只在 `step + yaw_drift` 上出现局部位置改善：
  - `rmse_position_mean = 1.0906` vs `uniform 1.1765`
  - `final_position_error_norm_mean = 13.1281` vs `14.2473`
- 但同一设置下 yaw 更差：
  - `rmse_yaw_mean = 1.0770` vs `0.8158`
- 在 `figure8 + yaw_drift` 和两个 `anisotropic_noise` 场景都没有构成优势。
- 这不是“有意义地打赢 `uniform`”，只能算 tradeoff，不足以支撑 milestone 3。

`xyz_only`

- 在所有聚合结果中都非常差。
- 例如：
  - `step + yaw_drift`: `rmse_position_mean = 169.5774`
  - `figure8 + anisotropic_noise`: `rmse_position_mean = 156.6631`
- 它是必须保留的 pruning control，但不是强 baseline。

## Milestone-3 Relevance

- milestone 3 要求：
  - 至少两个核心场景显著改善
  - 多随机种子下趋势稳定
- 本轮结果不满足。
- 当前最多只能说：
  - `robust_residual_stats` 在 `step + yaw_drift` 上出现位置项改善信号
  - 但它没有跨场景稳定，也没有维持 yaw 指标

## Did Any Candidate Beat Uniform?

- 严格按“有意义且稳定”的标准：`no`
- 按“某单项指标偶尔更好”的宽松标准：
  - `robust_residual_stats` 在 `step + yaw_drift` 的位置项略优
  - 但同一设置 yaw 更差，成功率也没有提升

## Did Any Candidate Beat xyz-only In At Least One Meaningful Case?

- `yes`, 但这个结论价值有限。
- `uniform`、`measurement_noise`、`robust_residual_stats` 基本都远好于 `xyz_only`。
- 问题在于 `xyz_only` 太弱，所以这不能支持“measurement-aware method 成立”。

## Failure Cases

- 所有 `anisotropic_noise` 聚合组的 `success_rate = 0.0`
- `step + yaw_drift` 的所有变体 `success_rate = 0.0`
- `figure8 + yaw_drift` 只有 `uniform` 和 `measurement_noise` 达到 `0.2`
- 所有运行都 `all_runs_finite = True`
  - 说明问题不是数值崩溃
  - 而是闭环质量本身达不到当前通过门槛

## Bottom Line

- 本轮聚合结果不支持 measurement-aware mainline 继续按 milestone 3 / 4 推进。
- 在当前冻结范围内，项目更像：
  - “四旋翼 DeePC 对 measurement heterogeneity 与 output structure 的敏感性分析”
- 而不是：
  - “一个已经站得住的新 measurement-aware 主方法”
