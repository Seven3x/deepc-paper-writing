# Direction Judge

最后更新时间：2026-04-01

这份裁判稿已经基于三个输入文件完成重读：

- [direction_literature_scan.md](/home/roxy/deepc-paper/paper/docs/status/direction_literature_scan.md)
- [direction_candidates.md](/home/roxy/deepc-paper/paper/docs/status/direction_candidates.md)
- [direction_method_sketches.md](/home/roxy/deepc-paper/paper/docs/status/direction_method_sketches.md)

## 判定标准

- `低撞题风险`：当前 DeePC + quadrotor / UAV 路线上，是否还能清晰讲成方法主线，而不是另一个已有路线的重复改写。
- `中等撞题风险`：相邻工作不少，但还没被完全占满。
- `高撞题风险`：很容易撞到已有路线，或很容易滑成平台工程 / 组合 trick。

## 先说明冲突

三份输入里有一处明显冲突：

- `direction_literature_scan.md` 把 `fault / degradation-aware DeePC` 判成 `A. 明显已被占用`。
- `direction_candidates.md` 和 `direction_method_sketches.md` 对同一方向给了更松的 `B` / `B`，理由是没查到明确的 quadrotor-DeePC 代表作。

我的保守处理是：

- 不采纳更乐观的 `B` 读法。
- 把 `fault / degradation-aware DeePC` 视为 `中等偏高撞题风险`，而不是低风险新主线。

这也是整份裁判稿里最重要的保守收缩点。

## 总体判断

当前项目不适合继续死磕旧主线，原因不是“测量问题不存在”，而是：

- 仓库内部结果已经把 `measurement-aware regularization` 的主线打成了不稳定候选，而不是成立的方法主张。
- 2021 年的四旋翼 DeePC 直接论文已经讨论过 noisy orientation、输出通道选择、weighted penalization 和后续 online update 的方向。
- 2025 年开始，`online data selection`、`contextual sampling`、`robust data selection`、`gain-scheduled DeePC` 这几条 DeePC 邻近路线都明显在收紧。

## 占坑判断

| 方向 | 已做程度 | 风险判断 | 保守理由 |
|---|---|---|---|
| `measurement-aware / covariance-aware regularization` | A | 高 | 2021 四旋翼 DeePC 已明确讨论 noisy orientation、discarding measurements、weighted penalization；仓库内部也已验证旧主线不稳。 |
| `online data-updated DeePC` | A | 高 | 2021 四旋翼 DeePC 已把 online Hankel update 写成 future work；2025 quadrotor online data selection 也把相邻路线继续往前推。 |
| `data selection / contextual sampling / computational efficiency` | A | 高 | 2025 `Select-DeePC`、`Contextual sampling`、`RDS-DeePC`、`Deep DeePC` 已经把这条线占得很满。 |
| `hyperparameter tuning / RL fine-tuning` | A/B | 高 | 这类工作最容易退化成调参论文，而且已经被 DeePC 近邻路线持续消耗创新空间。 |
| `regime-aware / maneuver-conditioned / multi-bank DeePC` | A | 高 | 2025 `Gain-Scheduled DeePC` 已经把 regime-conditioned 这条线显式打开；再往前做很容易变成局部 bank / scheduling heuristic。 |
| `fault / degradation-aware DeePC` | A/B | 中等偏高 | 直接 quadrotor-DeePC 代表作不够清晰，但四旋翼相关 DeePC 与 fault-tolerant 控制邻近工作已经很多；保守看不能当低风险新主线。 |
| `delay / dropout / asynchronous-measurement-aware DeePC` | B/C | 中等 | 我没有查到足够清晰的 quadrotor-DeePC 代表作；这条线更像结构性问题，而不是单纯权重微调。 |
| `safety-filtered / obstacle-aware DeePC` | B | 中等偏高 | 安全 / 障碍规避在 quadrotor 领域本身很拥挤，而且很容易依赖 CBF / corridor / MPC 组合，工程负担高。 |
| `multi-UAV / formation DeePC` | B | 高 | 多机编队、容错编队、分布式通信这几条线在 quadrotor 领域已很拥挤；再叠 DeePC 后工程复杂度很快上升。 |

## Top 3

### 1. Delay / Dropout / Asynchronous-Measurement-Aware DeePC

为什么入选：

- 这条线最像“单一主线 + 单一最小方法”。
- 它和当前 measurement-aware 主线不同，不是继续调 regularization，而是改观测链条本身。
- 对轻量仿真友好：可以用多频传感器、采样延迟、丢包、异步视觉 / 状态流直接构造。

为什么没排第一：

- benchmark 设计比看起来更敏感，容易滑向“数据管线论文”。
- 如果处理不严谨，审稿人会把它看成仿真时序问题，而不是 DeePC 方法贡献。
- 需要确保方法是结构性的，例如显式异步状态对齐或时延建模，而不是简单补齐缺失样本。

风险评估：

- 新意风险：低到中
- 实现风险：中
- 论文潜力：中到高

### 2. Fault / Degradation-Aware DeePC

为什么入选：

- 对 quadrotor 来说，单电机退化、执行器效率下降、传感器漂移都很自然。
- 轻量仿真里很容易做，不需要 Gazebo / PX4 / 实机。
- 如果只做一条主故障轴，方法可以保持单纯，不必依赖组合 trick。

为什么没排第一：

- 这条方向在输入文件里本身就有冲突：一个文件把它判成已明显占用，另外两个文件把它看得更空。我按更保守口径处理，所以不能给低风险评分。
- quadrotor fault-tolerant control 本身非常拥挤，容易被认为是传统 fault-tolerant control 的 DeePC 迁移版。
- 如果方法最后只是“故障时把某些通道权重调小”，会很快退化成另一个 tweak。

风险评估：

- 新意风险：中到高
- 实现风险：低
- 论文潜力：中

### 3. Regime-Aware / Maneuver-Conditioned DeePC

为什么入选：

- quadrotor 天然有 hover / turn / aggressive / near-ground 这类工况差异。
- 如果只保留一个调度变量或一个 regime 切换轴，方法主线可以写得很清楚。
- 轻量仿真可做，不需要重平台。

为什么没排第一：

- 这条路在 DeePC 邻近文献里最容易撞题。
- 2025 年已有 `Gain-Scheduled DeePC` 和 `Select-DeePC` 把 adjacent route 往前推。
- 很容易滑成 multi-bank heuristic，最后需要一堆 regime design 才能勉强成立。

风险评估：

- 新意风险：高
- 实现风险：中
- 论文潜力：中

## 明确不推荐

- `online data update`：已经太靠近已有 DeePC 邻近路线，且很容易退化成工程调度。
- `data selection / contextual sampling / computational efficiency`：已被 2025 代表作明显占坑，不适合当当前项目的新主线。
- `hyperparameter tuning / RL fine-tuning`：最容易变成调参论文，方法主张弱。
- `measurement-aware weighting tweak`：仓库内部已经把旧主线验证得不够稳，再继续做只会加深 tweak 化。
- `safety-filtered / obstacle-aware`：工程负担高，且容易依赖 CBF / corridor / MPC 组合，不适合当前“轻量仿真 + 方法主贡献”的约束。
- `multi-UAV / formation`：问题太大，工程与通信复杂度都高，不适合作为下一轮首发主线。

## 最终建议

下一轮先试的唯一主线：

- `Delay / Dropout / Asynchronous-Measurement-Aware DeePC`

备胎主线：

- `Fault / Degradation-Aware DeePC`

为什么这样排：

- `delay / async` 更像结构性方法问题，而不是另一个权重微调。
- `fault / degradation` 更容易落地，但当前输入文件已经表明它不该被当作低风险方向，只适合作为备胎。

## 下一轮前必须确认的 3 个问题

1. 这个方向能不能在当前轻量四旋翼底座里，用一个清晰的单轴非理想性讲完，而不是同时引入多个 nuisance factors？
2. 这个方向的最小方法，是否能只靠一个主机制成立，而不是依赖数据选择、额外观测器、CBF、bank switching 等组合 trick？
3. 这个方向的实验结果，是否能在轻量仿真里形成比“又一个 regularization tweak”更清晰的贡献边界？
