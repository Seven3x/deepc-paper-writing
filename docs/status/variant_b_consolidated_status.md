# Variant B Consolidated Status

最后更新时间：2026-04-01

## 输入文档

本汇总基于以下三份状态文件：

- [variant_b_failure_analysis.md](/home/roxy/deepc-paper/paper/docs/status/variant_b_failure_analysis.md)
- [variant_b_method_design.md](/home/roxy/deepc-paper/paper/docs/status/variant_b_method_design.md)
- [variant_b_experiment_status.md](/home/roxy/deepc-paper/paper/docs/status/variant_b_experiment_status.md)

## 1. Variant B 的核心靶点是什么

Variant B 的唯一靶点是：

**修复 Variant A 在短延迟场景下的全窗口过度对齐问题。**

更具体地说：

- Variant A 的失败模式不是“不会对齐”
- 而是“对整段历史窗口一刀切地强制对齐”
- 这在 `figure8 + delay_1` 这类短 delay、相位敏感轨迹里会过度校正

因此 Variant B 选择的唯一机制是：

**Delay-Sized Suffix Alignment DeePC**

也就是：

- 只对最近 `delay_steps` 个历史位置做对齐
- 更早的历史样本保留 received chronology
- 不叠 weighting、bank、delay estimation 或其他 trick

## 2. 它是否比 Variant A 更稳

**No**

实验结论是混合的：

- `step + delay_1`：Variant B 优于 A
- `figure8 + delay_2`：Variant B 优于 A
- `figure8 + delay_1`：Variant B 不如 A，也没有修复关键失败点
- `step + delay_2`：Variant B 不如 A

更关键的是：

- B 没有稳定降低 seed 方差
- B 没有把最关键失败模式扭转成明确正结果

所以不能说它“更稳”。

## 3. 它是否足以继续支撑 delay-aware 方法主线

**No**

理由：

- Variant A 只能支撑“部分场景有效”的 prototype 结论
- Variant B 是一次受控、低复杂度、单机制的最后保卫尝试
- 但它仍未把主张推进到“更稳的 delay-aware DeePC 方法”

到这里，方法线已经拿到了应有的信息增量，但没有拿到足够强的胜势。

## 4. 如果答案是否，是否应停止继续做 Variant C/D/E

**Yes**

理由：

- 当前最合理的低复杂度修正已经尝试过
- 再做 C/D/E 很容易演变成组合补丁和无约束打补丁
- 这不符合本轮“最后的小范围方法保卫尝试”的约束

因此，应该停止继续扩 Variant C / D / E。

## 5. 当前更合适的论文定位是什么

**B. prototype + analysis**

如果要再保守一点，也可以向：

**C. analysis / benchmark**

收缩。

不建议当前继续坚持：

**A. 继续方法论文主线**

## 建议措辞

当前最能被结果支撑的表述是：

- 标准 DeePC 在 fixed-delay 下会退化
- 单纯参考平移不能解释收益
- 历史窗口时间对齐本身有方法价值
- 但无论是 full-window alignment 还是 suffix-only alignment，都没有形成稳定普适赢家

## Bottom Line

- Variant B 的核心靶点是“短延迟下的过度全窗口对齐”。
- Variant B **没有**比 Variant A 更稳。
- 它 **不足以**继续支撑 delay-aware 方法主线。
- 应 **停止**继续做 Variant C / D / E。
- 当前最合适的论文定位是：**prototype + analysis**，必要时进一步收缩为 **analysis / benchmark**。
