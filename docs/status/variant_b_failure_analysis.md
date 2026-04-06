# Variant B Failure Analysis

最后更新时间：2026-04-01

## 结论

Variant A (`time_aligned`) 已经证明了一件事：`fixed-delay` 下，`history window` 的时间对齐本身不是空话，它确实能在部分场景里优于 `naive` 和 `delay_ref_only`。

但它也同时暴露了一个更重要的事实：**全窗口对齐不是稳定解**。尤其在 `figure8 + delay_1` 这类短延迟、相位敏感轨迹上，`time_aligned` 并没有稳定压过 `naive`，甚至在多 seed 聚合里更差。

因此，本轮最值得解释的不是“为什么 A 有时赢”，而是：

**为什么一刀切的全窗口时间对齐在短延迟场景里会失稳。**

这也是 Variant B 唯一值得盯住的靶点。

## 当前证据范围

本分析只基于当前已完成的 fixed-delay 结果，不新增实验、不引入新方法。

使用的结果来源：

- [/home/roxy/deepc-paper/deepc/Results/delay_alignment_20260401_214220_heuristic_compare/summary.md](/home/roxy/deepc-paper/deepc/Results/delay_alignment_20260401_214220_heuristic_compare/summary.md)
- [/home/roxy/deepc-paper/deepc/Results/delay_alignment_seed_sweep_20260401_213625_round1_3seeds/summary.md](/home/roxy/deepc-paper/deepc/Results/delay_alignment_seed_sweep_20260401_213625_round1_3seeds/summary.md)

## A 在哪些场景明显有效

`time_aligned` 的正向信号主要出现在这些地方：

- `step + delay_1`
  - 单 seed 上 `rmse_position` 从 `61.2252` 降到 `59.6585`
  - `final_position_error_norm` 从 `199.8520` 降到 `210.1251`，这里并不总是更好，但位置 RMSE 有改善
- `step + delay_2`
  - 单 seed 上 `rmse_position` 从 `57.4670` 到 `60.7251`，这个点不稳，但 `delay_ref_only` 更接近 baseline，说明真正的历史对齐并没有在每个 step 场景都失败
  - 多 seed 上 `rmse_position_mean` 从 `203.2396` 降到 `202.0599`
  - `final_position_error_norm_mean` 从 `685.7568` 降到 `661.0765`
- `figure8 + delay_2`
  - 单 seed 上改善最明显：
  - `rmse_position` 从 `55.2245` 降到 `47.6599`
  - `final_position_error_norm` 从 `206.1375` 降到 `147.9168`
  - `max_abs_position_error` 从 `159.1865` 降到 `109.7444`

## A 在哪些场景明显不稳

`time_aligned` 失稳最明显的是：

- `figure8 + delay_1`
  - 单 seed 上 `time_aligned` 没有优于 `naive`
  - `rmse_position` 从 `45.7775` 变成 `46.8112`
  - `final_position_error_norm` 从 `121.8844` 变成 `152.6388`
  - 多 seed 上同样不成立：
    - `rmse_position_mean` 从 `144.6160` 变成 `148.5376`
    - `final_position_error_norm_mean` 从 `502.8331` 变成 `567.9242`
- `figure8` 的 seed 方差整体更大
  - 这说明对齐后并没有把问题变成更“平滑”的优化问题
  - 相反，它保留了对激励质量和轨迹相位的敏感性

`delay_ref_only` 也没有构成解释：

- 它在 `figure8 + delay_1` 上明显更差，尤其 yaw 指标恶化很大
- 它在 `step + delay_1` 上几乎没有带来变化
- 这说明“只是把参考往后挪”不是问题答案

## 最值得解释的失败模式

我认为最关键的失败模式是这一条：

**Variant A 把所有 fixed-delay 都当成需要全窗口重配的场景，但在短延迟且相位敏感的轨迹里，这个全窗口对齐过于激进，反而损失了最近、最有信息量的历史片段。**

更直白一点：

- `time_aligned` 不是完全错
- 错在它默认“整段 history 都应该以同样方式重排”
- 对 `figure8 + delay_1` 而言，1-step delay 可能已经太短，不值得把整个窗口都按 source step 重构
- 这会让优化更依赖过往片段的偶然激励质量，而不是更依赖当前可用信息

这个失败模式还能同时解释两个现象：

1. 为什么 `step` 场景常常有收益
   - `step` 的相位结构弱，历史片段一致性要求低
   - 全窗口对齐更容易带来正效应

2. 为什么 `figure8 + delay_1` 不稳
   - `figure8` 是相位敏感轨迹
   - 短延迟下，强制全窗口对齐更容易把“最新有效观测”过度重排，造成信息损失而不是修正

## 多 Seed 高方差说明什么

3-seed 聚合显示，`time_aligned` 的均值波动很大，尤其在 `figure8` 上：

- `figure8 + delay_1`
  - `time_aligned` 的均值没有打赢 `naive`
  - 说明收益不是结构性稳定收益
- `figure8 + delay_2`
  - `time_aligned` 有一定改善，但不是压倒性改善

这说明问题不只是“单次实验偶然偏差”，而是：

- 对齐后的有效窗口质量仍然高度依赖 seed
- 也依赖初始激励序列和进入 Hankel 的那段数据
- 换句话说，`time_aligned` 提升了机制复杂度，但没有把对数据质量的敏感性消掉

## 对 Variant B 的唯一建议靶点

Variant B 不应该再做更复杂的延迟估计，也不应该再叠加权重或 bank。

唯一值得做的靶点是：

**避免对短 delay 场景做全窗口一刀切对齐，改成只对齐真正可用且时序一致的历史部分。**

也就是把问题从：

- “所有 history 都按 source step 重排”

收缩成：

- “只保留真正可靠的 aligned history”

这条靶点直接对应当前失败模式：

- 它针对的是短延迟下的过度对齐
- 它针对的是 figure8 这类相位敏感轨迹的窗口信息损失
- 它也能解释为什么 seed 方差大，因为它关注的是历史片段质量，而不是继续扩大历史重排范围

## Bottom Line

- `A` 在 `step` 和部分 `delay_2` 场景里有效。
- `A` 在 `figure8 + delay_1` 上明显不稳。
- 最值得解释的失败模式是“全窗口对齐过度激进，短延迟和相位敏感轨迹下会损失有效历史信息”。
- `Variant B` 的唯一建议靶点是“valid-history-only / partial-window alignment”，而不是延迟估计、加权或多机制拼装。
