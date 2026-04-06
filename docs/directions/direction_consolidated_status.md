# Direction Consolidated Status

最后更新时间：2026-04-01

## 1. 为什么当前项目不适合继续死磕旧主线

当前仓库已经有足够内部证据说明：

- `measurement-aware / covariance-aware regularized DeePC` 还没有形成可防守的新方法主张。
- 现有结果没有稳定击穿 `uniform regularized DeePC`。
- 这条线继续推进，很容易再次退化成：
  - 又一个 weighting tweak
  - 又一个 regularization 公式替换
  - 又一个超参数解释

更现实的判断是：

- 旧主线更适合收缩成 `analysis / sensitivity` 叙事。
- 如果还想找新的方法论文方向，必须换成一个**结构上不同于继续调 regularization** 的问题。

## 2. 已被明显占用的方向

以下方向当前不适合再当新主线：

- `measurement-aware / covariance-aware regularization`
  - 理由：四旋翼 DeePC 直接文献已经讨论 noisy orientation、measurement weighting、output pruning；仓库内部结果也没有支持继续硬推。
- `online data-updated DeePC`
  - 理由：`ODeePC`、recursive / closed-loop DeePC 及 2025 年相邻工作已经明显占坑。
- `data selection / contextual sampling / computational efficiency`
  - 理由：2025 年的 `Choose Wisely`、`Less is More`、以及 recursive / deep DeePC 已把这条线压得很紧。
- `regime-aware / maneuver-conditioned / multi-bank DeePC`
  - 理由：`Gain-Scheduled DeePC`、`Select-DeePC` 这类路线已经把工况切换 / local bank 的方法骨架占住。
- `hyperparameter tuning / RL fine-tuning`
  - 理由：最容易退化成调参论文，方法主张弱。
- 继续做 `measurement-aware weighting tweak`
  - 理由：仓库当前证据已经不支持再把它包装成新主线。

## 3. 候选新方向列表

### 低撞题风险

- 当前没有一个方向足够稳，可以直接标成低撞题风险。

### 中等撞题风险

- `Delay / Dropout / Asynchronous-Measurement-Aware DeePC`
  - 依据：我没有查到明确公开的 `quadrotor + DeePC + asynchronous measurement` 代表作；但它与 closed-loop / noisy-measurement / online DeePC 相邻，不能乐观。
- `Safety-filtered / Obstacle-aware DeePC`
  - 依据：DeePC 安全滤波已有相邻工作，quadrotor 避障本身也很拥挤；仍有方法空间，但工程负担偏高。

### 中等偏高撞题风险

- `Fault / Degradation-Aware DeePC`
  - 依据：subagent 结论有冲突；更保守的文献扫描把它判为已明显占用，相对乐观的候选和方法草图则认为 quadrotor-DeePC 直接代表作不够清晰。最终按更保守口径处理，只保留为备胎。

### 高撞题风险

- `Regime / Maneuver-Conditioned DeePC`
- `Online Data-Updated DeePC`
- `Data Selection / Contextual Sampling / Efficiency`
- `Hyperparameter Tuning / RL Fine-Tuning`
- `Measurement-Aware Weighting Tweak`
- `Multi-UAV / Formation DeePC`

高风险的共同原因是：

- 要么已被相邻工作明显占用；
- 要么工程复杂度高，不适合作为当前仓库的轻量仿真首发主线；
- 要么很容易滑成平台工程或组合 trick。

## 4. Top 3 排名

### 1. Delay / Dropout / Asynchronous-Measurement-Aware DeePC

- 入选原因：
  - 它改变的是观测链结构，不是继续调 `lambda` 或换 weighting 公式。
  - 适合当前轻量仿真底座，只需改测量流、buffer、`y_ini` 对齐和实验入口。
  - 最容易做成“单一主线 + 单一最小方法”。
- 风险：
  - 如果问题定义不够具体，会退化成“观测预处理小修小补”。
- 最小方法版本：
  - `mask/alignment-aware y_ini`
  - 固定延迟、随机丢包、低频单通道观测
  - 不上复杂观测器，不和别的方法组合
- 最小验证计划：
  - `step` / `figure8`
  - `nominal` / `fixed delay` / `random dropout` / `low-rate yaw`
  - 对比 `uniform`、当前 measurement-aware 版本、naive last-sample-hold
  - 先做 `3` seeds

### 2. Fault / Degradation-Aware DeePC

- 入选原因：
  - 在轻量仿真里容易注入 actuator efficiency drop、sensor drift 之类退化。
  - 比继续做 measurement-aware 更像一个独立问题。
- 为什么没排第一：
  - 文献占坑判断存在冲突，保守看不能当低风险新主线。
  - 若方法最后只是“按退化程度调权重”，会直接退化回旧主线的变种。
- 最小方法版本：
  - 两态 `nominal / degraded` bank 或单一 degradation state
  - 不做 fault isolation，不做复杂 classifier
- 最小验证计划：
  - `nominal` / `mild degradation` / `severe degradation`
  - `step` / `figure8`
  - 先做 `3` seeds

### 3. Regime / Maneuver-Conditioned DeePC

- 入选原因：
  - 与四旋翼任务本身匹配，工况区分自然。
- 为什么没排更高：
  - `GS-DeePC`、`Select-DeePC` 等已将这条方法骨架占住。
  - 很容易滑成 multi-bank heuristic。
- 最小方法版本：
  - 单一 scheduling variable
  - 一到两个 local Hankel bank
  - 不做复杂插值、混合专家或 RL gating
- 最小验证计划：
  - `hover` / `step` / `figure8` / one aggressive segment
  - 对比单 bank 和双 bank

## 5. 推荐先做的唯一主线

推荐下一轮优先尝试：

- `Delay / Dropout / Asynchronous-Measurement-Aware DeePC`

推荐理由：

- 它和旧主线结构上不同。
- 它不依赖 Gazebo / PX4 / 实机。
- 它可以在当前仓库里做出一个足够小、足够单纯的方法版本。
- 它目前是最接近“中等撞题风险，但仍可尝试”的方向。

## 6. 推荐的备胎主线

备胎主线：

- `Fault / Degradation-Aware DeePC`

保留原因：

- 轻量仿真落地快。
- 如果 delay / async 方向在最小版本上没有形成清晰贡献，它是最自然的下一条单轴问题。

降权原因：

- 文献占坑风险不能乐观。
- 容易退化成“故障时再调一次 regularization”。

## 7. 明确禁止现在就尝试的方法组合

以下组合当前不建议启动：

- `delay/async + fault/degradation` 组合
- `delay/async + safety filter` 组合
- `fault/degradation + regime-aware bank switching` 组合
- `regime-aware + online data selection` 组合
- `measurement-aware weighting + 任意新主线` 的拼接版
- `DeePC + CBF + bank switching + online update` 这类多组件堆叠

禁止原因：

- 当前目标是先找到一个**单一主线 + 单一最小方法**就能成立的方向。
- 组合方法会掩盖贡献边界，也会把工程复杂度迅速抬高。

## 8. 下一轮真正开发前必须先确认的 3 个问题

1. 新方向能否用一个单轴非理想性讲完，而不是一次性引入多个 nuisance factors？
2. 最小方法是否只靠一个核心机制就能成立，而不是依赖 observer、CBF、bank switching、online update 等组合件？
3. 最小实验矩阵能否在当前轻量仿真底座上跑出清晰边界，证明这不是“又一个 regularization tweak”？

## 9. 最终保守结论

这轮扫描没有找到一个可以乐观写成“低撞题风险、明显可成”的方向。

更保守、也更诚实的结论是：

- `Delay / Dropout / Asynchronous-Measurement-Aware DeePC` 值得先试；
- `Fault / Degradation-Aware DeePC` 可以保留作备胎；
- 其余方向当前不是撞题风险高，就是工程风险高，或者两者都高。

如果下一轮连 `delay / async` 的最小方法都讲不清楚，那么更合理的动作不是继续拼新 trick，而是停止开发新方法，回到 `analysis / benchmark` 定位。
