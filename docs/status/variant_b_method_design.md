# Variant B Method Design

最后更新时间：2026-04-01

## 1. Variant B 名称

**Delay-Sized Suffix Alignment DeePC**

简称：**Suffix-Aligned DeePC**

## 2. 核心机制

Variant B 只做一件事：

**不再对整个 `T_ini` 历史窗口做全量 time alignment，而是只对“延迟长度对应的最近后缀”做对齐，其余更早的历史样本保持原始收到顺序。**

在固定延迟 `d` 已知的前提下，历史窗口被拆成两段：

- 前缀：更早的历史样本，保留原始 received chronology
- 后缀：最近 `d` 个样本，按 fixed delay 做对齐

这不是增加一个新的估计器，也不是再叠加一个权重层，而是把 “全窗口统一平移” 改成 “只对最靠近当前控制时刻的延迟后缀进行对齐”。

## 3. 与 Variant A 的本质差异

Variant A 是 **full-window time alignment**：

- 参考窗口对齐
- 整个 `u_ini / y_ini` 历史窗口都按 delay 回退

Variant B 是 **suffix-only alignment**：

- 参考仍然按 fixed delay 对齐
- 但历史窗口只对最近的 delay 后缀做对齐
- 更早的历史样本不强行回退

本质上，A 假设“整个历史栈都应该等价地往过去挪”；B 假设“只有最靠近当前时刻、最容易受 delay 影响的部分需要对齐”，其余样本保留原始时间上下文。

## 4. 预期改善的失败模式

本轮只针对一个失败模式：

**短延迟下的过度校正。**

当前证据里最值得解释的反例是 `figure8 + delay_1`：

- 3-seed 汇总里，`time_aligned` 没有稳定优于 `naive`
- `delay_ref_only` 不能解释收益
- `time_aligned` 的 seed 方差仍然很大

这说明问题不一定是“不会对齐”，而更可能是：

**对整段历史窗口做统一回退，在短 delay / 相位敏感轨迹上会过头，导致有效动态信息被冲淡或错位。**

Suffix-only alignment 正是为这个失败模式设计的。

## 5. 最可能失败原因

最可能失败的原因有两个，但它们都指向同一个风险点：

- `figure8` 的相位敏感性比 `step` 高，单纯截断/局部对齐可能仍然不足以恢复一致性。
- 如果主要失配来自更早历史段，而不是最近后缀，那么 suffix-only alignment 会修正不够，收益会小于 Variant A。

换句话说，Variant B 可能失败于“修正太少”，而不是“机制太复杂”。

## 6. 最小代码改动点

如果后续真的实现，这个版本只需要改很少的地方：

- `Controllers/deepc.py`
- `run_experiment.py`

具体只改一条历史窗口构造路径：

- 在 `history_alignment` 中新增一个 suffix-aligned 分支
- 把 `u_ini / y_ini` 的填充逻辑拆成 prefix 和 suffix 两段
- suffix 段按 fixed delay 对齐，prefix 段保持原始 chronology

不需要：

- 新的 delay estimator
- 自适应切换
- 多个 heuristic 组合
- 独立的数据银行
- 额外的 residual weighting

## 7. 为什么值得做这一次最后的小冲刺

因为它是当前最小、最清楚、最可判别的修正。

现在已经知道：

- `naive` 不够
- `delay_ref_only` 解释不了收益
- `time_aligned` 说明“历史窗口对齐”本身有价值，但还不稳定

因此，最后一次方法冲刺最合理的方向不是继续加复杂度，而是测试一个更克制的假设：

**对齐应该只发生在最容易受 delay 污染的最近后缀，而不是强行覆盖整个历史窗口。**

如果这个版本还不能在 `figure8 + delay_1` 上比 Variant A 更稳，或者不能显著压低 seed 方差，那么就没有理由继续做 Variant C / D / E，应该停止方法线，回到 analysis / benchmark 叙事。

