# Fairness / Baseline Status

最后更新时间：2026-04-01 17:20

## 结论

- `xyz-only DeePC` 已经接入同一批量实验路径，不需要再补主入口 wiring。
- 但 `xyz-only` 目前不是强 baseline，而是一个需要保留的 pruning 对照。
- `LQR` / `linear MPC` 目前不构成 measurement-aware 主张下的公平对照，因为它们不消费被污染的测量路径。

## 当前状态

- `xyz-only` 可直接通过 [deepc/compare_deepc_regularization.py](/home/roxy/deepc-paper/deepc/compare_deepc_regularization.py#L104) 的 `--variants xyz_only` 运行。
- [run_experiment.py](/home/roxy/deepc-paper/deepc/run_experiment.py#L279) 已支持 `--output-set xyz`，所以 `xyz-only` 和 `xyzpsi` 在同一个实验入口里可切换。
- 目前没有发现需要为 `xyz-only` 新增专门 runner 的缺口。

## Controller x observation-path x corruption-path audit

| Controller | Observation path | Corruption path | Fair for main claim? | Comment |
| --- | --- | --- | --- | --- |
| `uniform regularized DeePC` | measured `y` | yes | yes | 主对照，和新方法共享同一路径。 |
| `measurement_noise DeePC` | measured `y` | yes | yes | 当前最合理的旧方法对照。 |
| `residual_stats DeePC` | measured `y` | yes | yes | 作为残差统计路线的对照。 |
| `xyz-only DeePC` | measured `y`, but only `xyz` outputs | yes | yes, but only as pruning control | 同路径但结构不同，不能当作“更强方法”，只能当作删通道对照。 |
| `LQR` | state feedback on `x` | no shared measurement corruption | no | 只能作为 nominal 工程基线，不是 measurement-aware robustness baseline。 |
| `linear MPC` | model-based state feedback on `x` | no shared measurement corruption | no | 同上。 |

## 证据摘要

- 在现有烟测里，`uniform DeePC` 在 `yaw_drift` 和 `anisotropic_noise` 下明显退化。
- `measurement_noise` 只在 `anisotropic_noise + step` 上出现有限改善，没形成稳定优势。
- `residual_stats` 没有超过 `measurement_noise`，且在 `yaw_drift` 上更差。
- `xyz-only` 不是强 baseline：已有结果里它在 nominal 下就可能明显偏弱，例如 `deepc_step_20260401_170614_xyz_nom_b` 的 `rmse_position = 3.0553`，远差于 `uniform DeePC` 的同类 nominal 结果。
- 另一个较温和的 pruning 跑法也没有证明 `xyz-only` 比 `uniform DeePC` 更强，只能说明它是必要的 pruning 对照，不是主贡献。
- `LQR` / `linear MPC` 在 [compare_measurement_scenarios.py](/home/roxy/deepc-paper/deepc/compare_measurement_scenarios.py#L207) 的脚本注释里已经明确写出测量扰动主要直接影响 DeePC，所以它们不属于 measurement-aware robustness 的同构基线。

## Valid baselines for the current claim

- `uniform regularized DeePC`
- `measurement_noise DeePC`
- `residual_stats DeePC`
- `xyz-only DeePC`，但要明确它是 pruning / ablation control，不是主方法

## Invalid or weak baselines

- `LQR`
- `linear MPC`
- `manual_grouped`
- `manual_yaw_only`
- `block_l2`
- `drop_yaw_past`

## Smallest repair plan

1. 保留 `LQR` / `linear MPC` 作为工程 baseline，只用于 nominal tracking 或平台 sanity check，不要把它们写成 measurement-aware robustness 的直接证据。
2. 在下一轮主实验里必须保留 `xyz-only`，否则无法判断方法收益是不是只是比 pruning 好一点。
3. 对 `xyz-only` 的实验定义要显式写清楚它使用的是更小输出集合，且当前实现里还伴随不同的数据收集设置；不要把它伪装成完全同构的控制器。
4. 只要还没有多种子聚合，就不要把单次 smoke 结果写成 paper-level claim。

## Recommendation

- `xyz-only` **必须**进入下一轮主实验矩阵。
- 但它应被定位为 negative control / pruning control，而不是强 baseline。
- 对主线写法，当前只能声称“measurement-aware regularization 相对 uniform DeePC 的改进趋势”，不能声称“已经全面压过所有基线”。
