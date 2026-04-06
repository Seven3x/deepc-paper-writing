# Method Agent Status

最后更新时间：2026-04-01 18:55

## 结论

这轮只做了 DeePC 一致性 / slack regularization 层的最小推进，没有改 `tracking cost Q`，没有引入复杂自适应框架。

我保留并核对了三类 residual-statistics 权重候选：

- `residual_variance`
- `residual_bias_variance`
- `robust_residual_stats`

结论很直接：

- 代码层，它们都比手工拍脑袋权重更可审计。
- 实验层，它们都还没有形成可重复的闭环优势。
- `uniform` 仍然是当前最稳的主参考线。
- `measurement_noise` 仍然是最稳的非均匀 baseline-like 策略，但它也没有在 frozen bad matrix 上稳定赢过 `uniform`。
- `residual_variance` 和 `robust_residual_stats` 在最新 shrinked smoke 里都没有打赢 `uniform`，而且在 `yaw_drift + step` 上明显更差。

## 本轮实际改动

- [/home/roxy/deepc-paper/deepc/Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)
- [/home/roxy/deepc-paper/paper/docs/status/method_agent_status.md](/home/roxy/deepc-paper/paper/docs/status/method_agent_status.md)

说明：

- `deepc.py` 里把四个 residual-statistics 函数统一到同一个校准残差矩阵入口，并加了维度检查。
- `run_experiment.py` 和 `compare_deepc_regularization.py` 本轮没有改逻辑，它们只是已经存在的比较入口。

## 精确公式

设 calibration rollout 收集到的残差为 `r_i(t)`，`i` 是输出通道，权重最终进入 `lambda_y * ||diag(w) sigma_y||_2`。

`residual_variance`

- `s_i = sqrt(var(r_i) + eps)`
- `w_i = clip( (1/s_i) / median_j(1/s_j), w_min, w_max )`

`residual_bias_variance`

- `mu_i = mean(r_i)`
- `s_i = sqrt(mu_i^2 + var(r_i) + eps)`
- `w_i = clip( (1/s_i) / median_j(1/s_j), w_min, w_max )`

`robust_residual_stats`

- `m_i = median(r_i)`
- `mad_i = median(|r_i - m_i|)`
- `robust_sigma_i = 1.4826 * mad_i`
- `s_i = sqrt(m_i^2 + robust_sigma_i^2 + eps)`
- `w_i = clip( (1/s_i) / median_j(1/s_j), w_min, w_max )`

Legacy `residual_stats` remains the older RMS-style rule:

- `s_i = sqrt(mean(r_i^2) + eps)`

## 为什么更 principled

- `residual_variance` 从通道残差能量估计可信度，比手工给 yaw 一个固定折扣更可解释。
- `residual_bias_variance` 把系统性偏置和随机波动一起纳入，理论上更贴近 `yaw_drift`。
- `robust_residual_stats` 用 MAD 类估计替代均方，对离群残差和非高斯扰动更稳。

这些都比经验手调更接近可复现的统计规则。

## Smoke Test

### 1) 最新 shrinked smoke

运行矩阵：

- trajectories: `step`, `figure8`
- scenarios: `nominal`, `yaw_drift`
- variants: `uniform`, `measurement_noise`, `residual_variance`, `robust_residual_stats`
- seed: `42`
- measurement seed: `0`

结果目录：

- [/home/roxy/deepc-paper/deepc/Results/deepc_reg_compare_20260401_184923_method_agent_smoke_v2](/home/roxy/deepc-paper/deepc/Results/deepc_reg_compare_20260401_184923_method_agent_smoke_v2)

命令：

```bash
MPLBACKEND=Agg /home/roxy/miniconda3/bin/conda run -n delivery python deepc/compare_deepc_regularization.py \
  --trajectories step,figure8 \
  --scenarios nominal,yaw_drift \
  --variants uniform,measurement_noise,residual_variance,robust_residual_stats \
  --reference-duration 6 --sampling-time 0.1 --dt 0.001 \
  --seed 42 --measurement-seed 0 --tag method_agent_smoke_v2
```

总计：

- `16` runs
- `14` runs 通过默认稳定阈值

关键结果：

- `nominal + step`
  - 四个变体完全一致：`rmse_position = 0.1483`, `rmse_yaw = 0.0430`
- `nominal + figure8`
  - 四个变体完全一致：`rmse_position = 0.0552`, `rmse_yaw = 0.0498`
- `yaw_drift + step`
  - `uniform` / `measurement_noise`: `rmse_position = 0.1403`, `rmse_yaw = 0.4162`, `final_position_error_norm = 0.0443`
  - `residual_variance` / `robust_residual_stats`: `rmse_position = 0.2017`, `rmse_yaw = 0.6709`, `final_position_error_norm = 1.3284`
- `yaw_drift + figure8`
  - `uniform` / `measurement_noise`: `rmse_position = 0.0588`, `rmse_yaw = 0.2805`, `final_position_error_norm = 0.1183`
  - `residual_variance` / `robust_residual_stats`: `rmse_position = 0.0647`, `rmse_yaw = 0.2894`, `final_position_error_norm = 0.3621`

### 2) Earlier broader sweep

我保留之前更大的 5-variant smoke 结论作为辅助证据：

- [/home/roxy/deepc-paper/deepc/Results/deepc_reg_compare_20260401_175139_method_sprint_v1](/home/roxy/deepc-paper/deepc/Results/deepc_reg_compare_20260401_175139_method_sprint_v1)

它补充说明了：

- `residual_bias_variance` 在 `anisotropic_noise + figure8` 上最不稳，不能当主候选。
- `measurement_noise` 是最稳的非均匀 baseline-like 权重。

## 失败模式

- `yaw_drift + step` 是当前最清楚的反例：`residual_variance` 和 `robust_residual_stats` 都比 `uniform` 差。
- `yaw_drift` 说明 calibration rollout 看到的早期 residual 不足以跟上漂移。
- `anisotropic_noise` 的更大矩阵里，残差统计候选也没有形成稳定超越 `uniform` 的趋势。

## 现在的排名

按当前可用证据排序：

1. `uniform`
2. `measurement_noise`
3. `residual_variance`
4. `robust_residual_stats`
5. `residual_bias_variance`

这个排序只表示当前 frozen matrix 下的综合可用性，不代表统计显著性。

## 推荐给主实验的候选

只保留两个候选给主实验 agent：

- `measurement_noise`
- `residual_variance`

理由：

- `measurement_noise` 是当前最稳的自动权重起点。
- `residual_variance` 是 residual-statistics 家族里最简单、最可审计的版本，虽然没有赢，但至少没有比更复杂的 `robust_residual_stats` 更差。

不建议把 `residual_bias_variance` 当主实验候选。它在更大的 sweep 里最不稳定。

## Bottom Line

- 这条方法线现在仍然是“有原则的权重设计”，但还不是“已经成立的新主方法”。
- 如果主实验 agent 继续推进，必须把它当成对 `uniform` 的小幅候选修正，而不是已经赢下的论文主张。
