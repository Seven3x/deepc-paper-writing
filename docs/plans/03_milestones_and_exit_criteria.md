# 里程碑与退出条件

## 里程碑 1：底座可用

目标：

- `DeePC_Quadcopter` 能稳定跑通基础轨迹和基础基线。

完成标准：

- `LQR / linear MPC / original DeePC` 跑通
- 结果可重复
- 指标可落盘

失败信号：

- 基线本身长期不稳定
- 同一设置重复运行波动过大

处理方式：

- 先修平台，不要急着做新方法

## 里程碑 2：问题被显式构造出来

目标：

- 在仿真里稳定构造出 `yaw drift / anisotropic noise / parameter mismatch` 导致的 DeePC 退化现象。

完成标准：

- `uniform DeePC` 在至少两个场景下明显恶化
- `xyz-only DeePC` 与 `full output` 有可解释差异

失败信号：

- 所有设置都差不多
- 说明问题没被造出来，或者当前输出结构不敏感

处理方式：

- 回头检查测量层与输出定义
- 不要盲目推进写作

## 里程碑 3：最小方法成立

目标：

- 第一版 `covariance-aware` 比 `uniform` 明显更稳。

完成标准：

- 至少两个核心场景显著改善
- 多随机种子下趋势稳定

失败信号：

- 只能偶尔赢
- 对协方差估计极端敏感

处理方式：

- 收缩为更简单的 group-wise 版本
- 或者退回把方法降级成辅助结论

## 里程碑 4：能打赢强 baseline

目标：

- 新方法不只是比原始 DeePC 好，还要能解释为什么不是简单 pruning 就够了。

必须比较：

- `original regularized DeePC`
- `xyz-only DeePC`
- `manual group-wise`
- `covariance-aware`

完成标准：

- 新方法至少在一类关键场景中优于 `xyz-only`
- 否则很难防守“你只是删变量”

## 里程碑 5：论文闭环

目标：

- 主结果、消融、失败案例、敏感性分析都齐全。

完成标准：

- 问题定义清楚
- 主假设被实验支持
- 审稿人可能的反驳已有对应实验

## 明确退出条件

如果出现下面任一情况，需要停止把它当主论文方向：

- `covariance-aware` 始终不如 `xyz-only`
- 改进只来自 tracking weight 调整，而不是 DeePC regularization
- 需要过于复杂的 adaptive 机制才有收益
- 仿真场景很难稳定重现论文核心现象

## 可接受的降级方案

如果主线不够强，可以降级为：

- “四旋翼 DeePC 输出结构与 regularization 敏感性分析”
- 把 `measurement-aware` 作为其中一个改进项，而不是整篇唯一方法贡献

## 当前建议的阶段顺序

1. 平台与 baseline
2. 测量模型层
3. 最小方法
4. 主实验
5. 强 baseline 对比
6. 消融与写作
