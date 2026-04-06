# Direction Candidates

最后更新时间：2026-04-01

这份文档只做保守候选筛选，不做乐观包装。

当前仓库内部已经把 measurement-aware 主线跑到“结果不支持继续硬推”的位置，所以这里不再把
`measurement-aware regularization tweak` 当新主线，而是只看还能不能找到一条单一主线、单一最小方法就能成立的替代路线。

## 1. 占坑总览

| 方向 | 已做程度 | 保守判断 |
| --- | --- | --- |
| fault / degradation-aware DeePC | B | 直接的 `quadrotor + DeePC + fault` 代表作我没有查到，但 `degradation-aware DeePC` 已经在能量系统里出现，quadrotor fault-tolerant control 也很拥挤。 |
| regime-aware / maneuver-conditioned DeePC | A | `Select-DeePC`、`contextual sampling`、`GS-DeePC` 已经把“按工况/区域/上下文选数据”这条线占得很满。 |
| delay / dropout / asynchronous-measurement-aware DeePC | B | 直接的 `quadrotor + DeePC + async measurement` 我没有查到明确代表作，但 `ODeePC`、`CL-DeePC`、online update 以及 delay-aware control 都已经很成熟。 |
| safety-filtered / obstacle-aware DeePC | A | 直接的 safety-critical DeePC 已经出现，quadrotor obstacle avoidance 也非常拥挤。 |
| multi-UAV / formation DeePC | B | 直接的 `multi-UAV + DeePC` 我没有查到明确代表作，但 distributed / decentralized DDPC 和 coordinated quadrotor control 已经很多。 |
| online data-updated / data selection / efficiency / hyperparameter tuning | A | `Select-DeePC`、`ODeePC`、`efficient recursive DeePC`、`Deep DeePC`、以及 2025 的 quadrotor online-DeePC 已经明显占坑。 |
| current measurement-aware regularization line | A | 仓库内部已经验证过这条线没有稳定击穿 `uniform`，也没有形成可防守的新主张。 |

### 直接依据的代表工作

- `Data-enabled predictive control for quadcopters` 里的 yaw bias / yaw drift / measurement covariance 讨论，和本仓库当前 measurement-aware 主线高度重叠。
- `Online Data-Enabled Predictive Control for Quadrotor Trajectory Tracking` 已经把 quadrotor online update 写成 2025 的明确方向，而且平台还是 ROS+PX4 SITL，不符合本仓库“不把平台工程当主贡献”的约束。
- `Choose Wisely: Data-Enabled Predictive Control for Nonlinear Systems Using Online Data Selection`、`Less is More: Contextual Sampling for Nonlinear Data-Enabled Predictive Control`、`Gain-Scheduling Data-Enabled Predictive Control for Nonlinear Systems with Linearized Operating Regions` 已经把数据选择、上下文采样、区域切换这条线占得很满。
- `Efficient Recursive Data-enabled Predictive Control` 和 `Deep DeePC` 已经把“效率/递推/替代在线优化”这条线占得很满。
- `Degradation-aware data-enabled predictive control of energy hubs` 说明 degradation-aware DeePC 本身不是空白。
- `Safety-critical data-enabled predictive control for wheeled mobile robot` 说明 safety-critical DeePC 本身也不是空白。

## 2. 候选方向

### 2.1 Delay / Dropout / Asynchronous Measurement-aware DeePC

- 一句话定义：把四旋翼 DeePC 的核心难题从“测量噪声权重怎么调”改成“观测不同步、丢包、延迟、低频/高频混采时，如何构造可对齐的 past window 和 Hankel 约束”。
- 最接近已做工作：`ODeePC`、`CL-DeePC`、当前仓库里已经接入的 yaw drift / anisotropic noise / measurement logging；我没有查到明确公开的 `quadrotor + DeePC + asynchronous measurement` 代表作。
- 新意风险：中
- 实现风险：中
- 论文潜力：中
- 最小方法版本：只做一个 lag-aware buffer，不上大模型、不做复杂融合。核心是为每个输出通道保留时间戳，允许不同通道的 `y_ini` 由最近可用观测、短时 hold、或固定延迟补齐，然后在 DeePC 里对缺失/滞后通道做 mask，而不是把它们直接当成噪声加权问题。
- 最小实验矩阵：`step` / `figure8` x `nominal` / `fixed delay` / `random dropout` / `low-rate yaw`，先用 3 个 seed；对比 `uniform`、当前 measurement-aware 版本、和 naive last-sample-hold。
- 为什么比继续做 measurement-aware 更值得试或不值得试：更值得试，因为它改的是信息到达结构，不只是换权重；但前提是你必须把“异步对齐”写成方法核心，否则很容易退化成观测预处理小修小补。

### 2.2 Fault / Degradation-aware DeePC

- 一句话定义：让 DeePC 显式面对 actuator efficiency loss、单电机退化、传感器漂移或部分故障，而不是只在正常系统上调 regularization。
- 最接近已做工作：`Degradation-aware data-enabled predictive control of energy hubs`、quadrotor fault-tolerant control 的一堆邻近工作（例如 2021 NMPC、2025 learning-based PFTC）；本仓库现有的 yaw bias / yaw drift 也可以作为最小退化场景，但那一层仍然更像 measurement corruption。
- 新意风险：中
- 实现风险：中
- 论文潜力：中
- 最小方法版本：只引入一个 degradation state 或一个通道效率向量，例如 `eta_u` 或 `eta_y`，然后让 DeePC 的 slack / regularization / input map 随这个退化量切换。不要一上来做 fault detection + fault isolation + reconfiguration 三件套。
- 最小实验矩阵：`nominal`、`mild degradation`、`severe degradation`，外加 `single-rotor efficiency drop` 或 `sensor bias drift`；先用 `step` 和 `figure8`，3 个 seed 足够看趋势。
- 为什么比继续做 measurement-aware 更值得试或不值得试：如果你能把退化量定义得足够物理，这条线比 measurement-aware 更像“新问题”；但如果最后只是把退化也写成一组权重，它就会退化回另一个 tweak。

### 2.3 Safety-filtered / Obstacle-aware DeePC

- 一句话定义：用 DeePC 负责 nominal tracking，再在外面加一个极薄的 safety filter 或 CBF / safe-corridor 层，专门处理障碍物和越界。
- 最接近已做工作：`Safety-critical data-enabled predictive control for wheeled mobile robot`、`Safe Quadrotor Navigation using Composite Control Barrier Functions`、以及大量 quadrotor obstacle avoidance MPC / CBF 工作。
- 新意风险：高
- 实现风险：高
- 论文潜力：中
- 最小方法版本：只做一层 one-step QP safety filter，不做完整规划器，不做地图搜索，不做平台工程。DeePC 输出 nominal command，filter 只负责把它投影回安全集。
- 最小实验矩阵：free flight、single static obstacle、narrow corridor、one dynamic obstacle；用 success rate、collision rate、tracking degradation 作为主指标，seed 不需要太多但要有至少 3 次重复。
- 为什么比继续做 measurement-aware 更值得试或不值得试：只有在你能给出清楚的硬安全优势时才值得试；否则它很容易变成“DeePC + 一个安全 wrapper”，方法主张不强，而且和 quadrotor obstacle avoidance 的既有工作重叠很大。

### 2.4 Multi-UAV / Formation DeePC

- 一句话定义：把单机 DeePC 扩展到两机或多机编队，让耦合约束、相对距离、通信延迟进入同一个 predictive control 框架。
- 最接近已做工作：`Decentralized Data-Enabled Predictive Control for Power System Oscillation Damping`、`Decentralized Robust Data-driven Predictive Control for Smoothing Mixed Traffic Flow`、以及大量 coordinated quadrotor flight / swarm MPC 工作；我没有查到明确公开的 `multi-UAV formation + DeePC` 代表作。
- 新意风险：中
- 实现风险：高
- 论文潜力：中
- 最小方法版本：先别做分布式，先做 2 机 centralized formation DeePC，只有一个 formation error penalty 和一个 collision margin。把通信、分布式求解、异步一致性都放到后续。
- 最小实验矩阵：2 机编队直线、8 字、变间距跟随；再加一个简单的 communication delay / packet loss 版本；和单机 DeePC 做对照。
- 为什么比继续做 measurement-aware 更值得试或不值得试：如果你想要“明显不同于当前主线”的故事，它确实不同；但对当前仓库来说，它更像新系统课题而不是轻量方法论文，工程代价和协同复杂度都偏高。

### 2.5 Regime-aware / Maneuver-conditioned DeePC

- 一句话定义：按 hover、turn、aggressive segment、near-ground、load-change 等工况分段，给每个工况一套 DeePC bank 或 scheduler。
- 最接近已做工作：`Select-DeePC`、`contextual sampling`、`GS-DeePC`、`closed-loop DeePC` 以及当前仓库已经明确降权的 mode-aware / multi-bank 方向。
- 新意风险：高
- 实现风险：中
- 论文潜力：低到中
- 最小方法版本：如果一定要做，只能保留一个 measurable scheduler variable，例如曲率、速度、或高度，然后切一到两个 Hankel bank；不要把它扩成 bank ensemble、混合专家或 RL gating。
- 最小实验矩阵：`hover` / `step` / `figure8` / one aggressive segment，比较单 bank 和双 bank；再看 switching 是否稳定。
- 为什么比继续做 measurement-aware 更值得试或不值得试：不值得作为首发主线。这个 lane 已经被在线数据选择、上下文采样、区域切换占得很满，很容易被看成又一个 bank-selection tweak。

### 2.6 Online Data-updated / Data-selection / Efficiency / Hyperparameter-tuning DeePC

- 一句话定义：在线更新 Hankel、做 contextual sampling、做 fixed-budget selection、做 recursive SVD update，或者用自动搜索 / RL 去挑 DeePC 超参数。
- 最接近已做工作：`Online Data-Enabled Predictive Control for Quadrotor Trajectory Tracking`、`Efficient Recursive Data-enabled Predictive Control`、`Deep DeePC`、`Select-DeePC`、`contextual sampling`，以及本仓库当前已经做过的 measurement-aware 调参探索。
- 新意风险：高
- 实现风险：中
- 论文潜力：低
- 最小方法版本：如果非做不可，只能保留一个非常干净的 recursive update 或 time-windowed Hankel 版本；但这已经很接近已有工作，且平台感很强。
- 最小实验矩阵：runtime vs tracking error 的双轴对比；但这条线在本仓库里会天然碰到“不把平台工程当主贡献”的硬约束。
- 为什么比继续做 measurement-aware 更值得试或不值得试：不值得。它要么撞上已有工作，要么沦为效率/实现细节论文；而本仓库明确不打算把平台工程当主贡献。

## 3. 先过滤掉的方向

下面这些方向不是“绝对不能做”，但按当前证据和仓库约束，我不建议把它们当下一轮主线：

- `regime-aware / maneuver-conditioned DeePC`
- `online data-updated / data selection / efficiency / hyperparameter tuning`
- `multi-UAV / formation DeePC`

原因很直接：

- 这几条里有两条已经明显被相邻工作占满，剩下一条工程复杂度高、很容易把时间花在 coordination 细节上。
- 如果硬做，最容易退化成 `analysis / benchmark / implementation note`，而不是一篇结构清晰的方法论文。

## 4. 当前最保守的读法

如果只看“新方法主线”而不看已有平台积累，这个仓库最像还能继续推进的方向只有两类：

1. `delay / dropout / asynchronous measurement-aware DeePC`
2. `fault / degradation-aware DeePC`

其余方向要么已经明显被占坑，要么工程负担和撞题风险都偏高。

## 5. Sources Used Most Directly

- [Data-enabled predictive control for quadcopters](https://pmc.ncbi.nlm.nih.gov/articles/PMC9291934/)
- [Online Data-Enabled Predictive Control for Quadrotor Trajectory Tracking](https://colab.ws/articles/10.23919%2Fccc64809.2025.11179176)
- [Online data-enabled predictive control](https://www.sciencedirect.com/science/article/pii/S0005109821004507)
- [Closed-loop Data-enabled Predictive Control and its equivalence with Closed-loop Subspace Predictive Control](https://arxiv.org/abs/2402.14374)
- [Choose Wisely: Data-Enabled Predictive Control for Nonlinear Systems Using Online Data Selection](https://arxiv.org/abs/2503.18845)
- [Less is More: Contextual Sampling for Nonlinear Data-Enabled Predictive Control](https://arxiv.org/abs/2503.23890)
- [Gain-Scheduling Data-Enabled Predictive Control for Nonlinear Systems with Linearized Operating Regions](https://arxiv.org/abs/2512.02797)
- [Degradation-aware data-enabled predictive control of energy hubs](https://arxiv.org/abs/2307.01543)
- [Safety-critical data-enabled predictive control for wheeled mobile robot](https://www.sciencedirect.com/science/article/pii/S0019057826000492)
- [Efficient Recursive Data-enabled Predictive Control](https://arxiv.org/abs/2309.13755)
- [Deep DeePC: Data-enabled predictive control with low or no online optimization using deep learning](https://arxiv.org/abs/2408.16338)
- [Current repository status: parent consolidated status](/home/roxy/deepc-paper/paper/docs/status/parent_consolidated_status.md)
- [Current repository status: experiment agent status](/home/roxy/deepc-paper/paper/docs/status/experiment_agent_status.md)
