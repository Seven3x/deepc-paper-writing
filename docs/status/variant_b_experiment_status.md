# Variant B Experiment Status

最后更新时间：2026-04-01

## Variant B

本轮唯一 Variant B 是：

**Delay-Sized Suffix Alignment DeePC**

实现名：

- `suffix_aligned`

核心机制只有一个：

- 历史窗口不再做 full-window time alignment
- 只对最近 `delay_steps` 长度的后缀做对齐
- 更早的 prefix 保持 received chronology
- 参考窗口仍按 fixed delay 对齐

## 实际运行矩阵

冻结矩阵已按要求执行：

- trajectories: `step`, `figure8`
- delay scenarios: `delay_1`, `delay_2`
- seeds: `41, 42, 43`
- controllers:
  - `naive`
  - `delay_ref_only`
  - `time_aligned`
  - `suffix_aligned`

结果目录：

- [delay_alignment_seed_sweep_20260401_222513_variant_b_round1](/home/roxy/deepc-paper/deepc/Results/delay_alignment_seed_sweep_20260401_222513_variant_b_round1)

聚合文件：

- [aggregate.csv](/home/roxy/deepc-paper/deepc/Results/delay_alignment_seed_sweep_20260401_222513_variant_b_round1/aggregate.csv)
- [summary.md](/home/roxy/deepc-paper/deepc/Results/delay_alignment_seed_sweep_20260401_222513_variant_b_round1/summary.md)

## 摘要结果

### figure8 + delay_1

- `naive`
  - `rmse_position_mean = 144.6160`
  - `final_position_error_norm_mean = 502.8331`
  - `rmse_yaw_mean = 25.7755`
- `time_aligned`
  - `rmse_position_mean = 145.6881`
  - `final_position_error_norm_mean = 530.3734`
  - `rmse_yaw_mean = 24.7023`
- `suffix_aligned`
  - `rmse_position_mean = 152.5020`
  - `final_position_error_norm_mean = 596.4984`
  - `rmse_yaw_mean = 23.5499`

结论：

- Variant B 没有修复 `figure8 + delay_1`
- 它的 yaw 均值略低，但位置指标更差
- 这个最关键失败场景没有被救回来

### figure8 + delay_2

- `naive`
  - `rmse_position_mean = 154.2270`
  - `final_position_error_norm_mean = 624.7694`
- `time_aligned`
  - `rmse_position_mean = 150.3138`
  - `final_position_error_norm_mean = 549.1761`
- `suffix_aligned`
  - `rmse_position_mean = 146.9651`
  - `final_position_error_norm_mean = 524.0251`

结论：

- Variant B 在这个场景下优于 Variant A
- 位置指标改善最明显
- 但 yaw 指标没有同步改善

### step + delay_1

- `naive`
  - `rmse_position_mean = 206.3237`
  - `final_position_error_norm_mean = 718.0071`
- `time_aligned`
  - `rmse_position_mean = 206.8750`
  - `final_position_error_norm_mean = 753.3743`
- `suffix_aligned`
  - `rmse_position_mean = 199.6317`
  - `final_position_error_norm_mean = 635.7510`

结论：

- Variant B 在这个场景下是最好的
- 比 A 更稳，也比 naive 更好

### step + delay_2

- `naive`
  - `rmse_position_mean = 203.2396`
  - `final_position_error_norm_mean = 685.7568`
- `time_aligned`
  - `rmse_position_mean = 201.3334`
  - `final_position_error_norm_mean = 639.9056`
- `suffix_aligned`
  - `rmse_position_mean = 208.3039`
  - `final_position_error_norm_mean = 764.5525`

结论：

- Variant B 在这个场景下明显不如 Variant A
- 也不如 naive

## Seed 间波动

从 `aggregate.csv` 的标准差看：

- `figure8 + delay_1`
  - `time_aligned` 的 `rmse_position_std = 72.6335`
  - `suffix_aligned` 的 `rmse_position_std = 72.3666`
  - 方差没有本质下降
- `figure8 + delay_2`
  - `time_aligned` 的 `rmse_position_std = 71.3220`
  - `suffix_aligned` 的 `rmse_position_std = 66.0494`
  - 有一定改善，但不大
- `step + delay_1`
  - `time_aligned` 的 `rmse_position_std = 106.2445`
  - `suffix_aligned` 的 `rmse_position_std = 100.1926`
  - 略有改善
- `step + delay_2`
  - `time_aligned` 的 `rmse_position_std = 104.4931`
  - `suffix_aligned` 的 `rmse_position_std = 110.1251`
  - 反而变差

结论：

- Variant B 没有把 seed 方差稳定压下去
- 它只是在部分场景有改善，不构成“整体更稳”

## Success / Pass Rate

当前 runner 没有预置本轮专用的任务阈值型 pass criterion。

在已有可用指标下：

- 所有 24 个运行都完成了，`all_finite = true`
- 因此按“数值稳定完成”定义的 finite pass rate：
  - `naive`: `100%`
  - `delay_ref_only`: `100%`
  - `time_aligned`: `100%`
  - `suffix_aligned`: `100%`

这说明：

- 本轮区分方法优劣的关键不是是否数值发散
- 而是位置误差和 seed 波动是否更稳

## 硬判断

### B 是否比 A 更稳

**No**

原因：

- `step + delay_1`：B 更好
- `figure8 + delay_2`：B 更好
- `figure8 + delay_1`：B 更差
- `step + delay_2`：B 更差

最关键的是：

- B 没有修复本轮最应修复的 `figure8 + delay_1`
- B 也没有稳定降低 seed 方差

### B 是否值得继续

**No**

原因：

- 它不是稳定赢家
- 它没有把最关键失败模式扭转成正结果
- 它会把方法叙事继续推向“场景依赖的小修补”

### 若不值得继续，是否应停止方法主线

**Yes**

原因：

- Variant A 已经证明“历史窗口对齐”本身有方法价值
- Variant B 已经做了最后一次低复杂度、单机制保卫尝试
- 但它仍然不能把结论推进到“更稳的 fixed-delay 方法”

## Bottom Line

- Variant B 不是空改动，它在部分场景确实优于 A。
- 但它没有比 Variant A 更稳。
- 它没有修复最关键的 `figure8 + delay_1` 失败点。
- 因此本轮应停止继续做 Variant C / D / E，回到 `prototype + analysis` 或 `analysis / benchmark` 叙事。
