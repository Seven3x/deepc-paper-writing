# Direction Literature Scan

最后更新时间：2026-04-01

范围：
- `DeePC`
- `quadrotor / UAV`
- `lightweight simulation`
- 明确排除 `Gazebo / PX4 / real hardware` 作为主评估场景

说明：
- 这里的 `A / B / C` 只表示公开文献占坑程度，不表示方法好坏。
- `A` = 明显已被占用
- `B` = 相邻工作较多但未明显占满
- `C` = 我没有查到明确公开代表作
- 我尽量只用一手来源或作者页；遇到从相邻路线推断的结论，会显式写 `Inference:`

## 1. fault / degradation-aware DeePC

- 已做程度：`A. 明显已被占用`
- 依据：
  - 直接的 quadrotor DeePC 代表作已经明确做了“系统退化 / yaw 校准误差”方向的比较，见 [Data-Driven Control Strategies for Rotary Wing Aerial Vehicles](https://novaresearch.unl.pt/en/publications/data-driven-control-strategies-forrotary-wing-aerial-vehicles/) (Robot 2023 / Springer 2024)。
  - 更早的 quadrotor DeePC 工作也已经把“非线性 + 噪声测量 + 正则化”作为核心问题，见 [Data-Enabled Predictive Control for Quadcopters](https://www.research-collection.ethz.ch/items/daced287-b2be-4efa-b089-bd0cf9ed678f) (ETH, 2019; working paper/2019 version)。
  - 更泛的 `degradation-aware DeePC` 也已经在别的系统里出现，见 [Degradation-aware data-enabled predictive control of energy hubs](https://arxiv.org/abs/2307.01543) (2023)。
- 保守判断：
  - `Inference:` quadrotor 上“单电机退化 / 传感器偏置 / drift / calibration error”这一类问题，已经很难再被写成一个全新方向，而更像是已被占用的 DeePC 鲁棒性分支。
  - `Inference:` 如果把“fault”限定到真正的 rotor failure / actuator loss-of-effectiveness，公开 DeePC 代表作我没有查到得足够清楚，但这条线在 quadrotor FTC 里本身就非常拥挤，不适合作为低撞题主线。

## 2. regime-aware / maneuver-conditioned DeePC

- 已做程度：`A. 明显已被占用`
- 依据：
  - `gain-scheduled DeePC` 已经直接出现，见 [Gain-Scheduled Data-Enabled Predictive Control for Nonlinear Systems with Linearized Operating Regions](https://arxiv.org/abs/2512.02797) (2025)。
  - `regime-varying / locally linear data representation` 的 DeePC 也已经明确写成方法名，见 [Gain-Scheduled Data-Enabled Predictive Control: A DeePC Approach for Nonlinear Systems](https://arxiv.org/abs/2509.26334) (2025)。
  - `NPV-DeePC / rapidly changing conditions` 也已经出现在后续 robust DeePC 体系里，见 [Robust Data-driven Predictive Control of Nonlinear Systems Under Model Uncertainty](https://open.clemson.edu/all_dissertations/4154/) (2025 dissertation)。
- 保守判断：
  - `Inference:` 这一类方法不一定已经在 quadrotor 上被写满，但 `DeePC + measurable regime / scheduling variable / local Hankel` 这条方法骨架已经被占住了。
  - 如果再往 quadrotor 里套 `hover / turn / aggressive / near-ground` 之类工况切换，容易被看成“把已知 gain-scheduling DeePC 换一个载体”。

## 3. delay / dropout / asynchronous-measurement-aware DeePC

- 已做程度：`B. 相邻工作较多但未明显占满`
- 依据：
  - `Online data-enabled predictive control` 已经处理实时测量反馈和系统变化，见 [Online data-enabled predictive control](https://www.sciencedirect.com/science/article/pii/S0005109821004507) (Automatica 2021)。
  - `Closed-loop DeePC / IV` 已经明确把 noise-induced bias 当问题处理，见 [Data-enabled predictive control with instrumental variables](https://arxiv.org/abs/2209.05210) (2022) 和 [Closed-loop Data-Enabled Predictive Control and its equivalence with Closed-loop Subspace Predictive Control](https://arxiv.org/abs/2402.14374) (2024)。
  - `Data-enabled predictive control for quadcopters` 已经强调每次实验都重采集数据并且正则化对噪声很关键，见 [Data-Enabled Predictive Control for Quadcopters](https://www.research-collection.ethz.ch/items/daced287-b2be-4efa-b089-bd0cf9ed678f) (2019)。
- 保守判断：
  - `Inference:` 我没有查到一个足够清晰的公开代表作，专门把 `dropout / missing samples / asynchronous streams / visual low-rate + IMU high-rate` 作为 quadrotor DeePC 主问题来写。
  - 但这条线离 `measurement noise / closed-loop bias / online adaptation` 太近，很容易继续退化成“又一个观测处理 tweak”。

## 4. safety-filtered / obstacle-aware DeePC

- 已做程度：`B. 相邻工作较多但未明显占满`
- 依据：
  - `DeePC + safety filter / CBF` 这条线已经开始在其他机器人上出现，见 [Safety-critical data-enabled predictive control for wheeled mobile robot](https://www.sciencedirect.com/science/article/abs/pii/S0019057826000492) (2026)。
  - quadrotor 上的安全/避障相关工作非常多，但大多是 `CBF + MPC / RL / DPC`，不是 DeePC 主线，例如 [Differentiable Predictive Control for Robotics: A Data-Driven Predictive Safety Filter Approach](https://arxiv.org/abs/2409.13817) (2024) 的 quadcopter demo 说明“安全滤波”本身已经是拥挤方向。
  - quadrotor 的障碍/编队避障也已有大量 MPC / CBF / consensus 路线，例如 [Obstacles avoidance for quadrotor formation based on consensus theory and S-MPC](https://www.sys-ele.com/EN/10.12305/j.issn.1001-506X.2024.02.29) (2024)。
- 保守判断：
  - `Inference:` 这里不是“没人做过”，而是“DeePC 作为主方法的公开占位还没把 quadrotor 避障这块完全钉死”。
  - 工程负担高，且很容易把论文做成 `CBF/MPC` 平台工程，而不是 DeePC 方法论文。

## 5. multi-UAV / formation DeePC

- 已做程度：`C. 我没有查到明确公开代表作`
- 依据：
  - 我检索到很多 multi-UAV / formation / FTCC / consensus / CBF 方向，但没有查到一个足够清楚、可直接引用的 `DeePC + multi-UAV formation` 公开代表作。
  - 现有相邻路线主要是 `RL formation`, `MPC formation`, `fault-tolerant formation`, `consensus + CBF`，例如 [Fault-Tolerant Cooperative Control of Unmanned Aerial Vehicles](https://link.springer.com/book/10.1007/978-981-99-7661-4) 这类书和大量 formation/MPC 文献。
- 保守判断：
  - 这条线看起来未被 DeePC 直接占满，但工程复杂度和论文风险都偏高。
  - `Inference:` 如果没有明确的分布式 / 层级式 DeePC 结构，最后很容易变成“多机仿真平台 + 复杂协调策略”，方法主张不够干净。

## 6. regularized DeePC

- 已做程度：`A. 明显已被占用`
- 依据：
  - 经典 quadrotor DeePC 工作已经明确指出 regularization 是必要的，并给出调参指导，见 [Data-Enabled Predictive Control for Quadcopters](https://www.research-collection.ethz.ch/bitstreams/5972b59c-f24c-4281-90c7-a5670f226763/download) (ETH working paper, 2019/2022 publication line)。
  - 这篇工作已经把 `regularized DeePC`、`noise`、`hyperparameter sensitivity` 作为核心内容，不是边角料。
- 保守判断：
  - 如果只是在 regularization 上再加一个权重项、再换一个范数、再调一个 penalty，撞题风险非常高。

## 7. online data-updated DeePC

- 已做程度：`A. 明显已被占用`
- 依据：
  - [Online data-enabled predictive control](https://www.sciencedirect.com/science/article/pii/S0005109821004507) (Automatica 2021) 已经明确提出 ODeePC。
  - [Efficient Recursive Data-enabled Predictive Control (Extended Version)](https://arxiv.org/abs/2309.13755) (2023) 已经把 recursive update / fast SVD update 做成主线。
  - [Closed-loop Data-Enabled Predictive Control and its equivalence with Closed-loop Subspace Predictive Control](https://arxiv.org/abs/2402.14374) (2024) 又进一步把 closed-loop / IV / sequential predictors 这条线做实。
- 保守判断：
  - 这条线已经不是“新方向”，更像 DeePC 的工程化支线。
  - 在 quadrotor 上继续做 online update，除非有非常强的系统约束，否则很难摆脱“计算加速 / 自适应修补”的叙事。

## 8. data selection / contextual sampling

- 已做程度：`A. 明显已被占用`
- 依据：
  - [Data selection and data-enabled predictive control for a fuel cell system](https://www.sciencedirect.com/science/article/pii/S2405896323022449) (2023) 已经把列选择、sRRQR、leverage score 这些数据选择策略系统比较过。
  - [Less is More: Contextual Sampling for Nonlinear Data-Enabled Predictive Control](https://arxiv.org/abs/2503.23890) (2025) 已经直接提出 contextual sampling 作为 DeePC 的数据选择新主线。
  - [Choose Wisely: Data-Enabled Predictive Control for Nonlinear Systems Using Online Data Selection](https://arxiv.org/abs/2503.18845) (2025) 也已经把 online data selection 写成方法名。
- 保守判断：
  - `Inference:` 如果再做 quadrotor 上的固定预算数据筛选，很容易被看成“把已有 data selection 文献搬到 UAV”。
  - 这条线撞题风险高，尤其是只做 `leverage score / sRRQR / contextual sampling` 的对比时。

## 9. hyperparameter tuning / RL fine-tuning

- 已做程度：`B. 相邻工作较多但未明显占满`
- 依据：
  - quadrotor DeePC 已经做过明确的 hyperparameter sensitivity study，见 [Data-Driven Control Strategies for Rotary Wing Aerial Vehicles](https://novaresearch.unl.pt/en/publications/data-driven-control-strategies-forrotary-wing-aerial-vehicles/) (2024)。
  - 更早的 quadrotor DeePC 论文也已经把 `T_d / T_ini / lambda_g` 等调参规律当作贡献，见 [Data-Enabled Predictive Control for Quadcopters](https://www.research-collection.ethz.ch/items/daced287-b2be-4efa-b089-bd0cf9ed678f)。
  - 更广义的 RL fine-tuning / auto-tuning 在控制领域是老路，但我没有查到一个足够清晰的 `DeePC + quadrotor + RL hyperparameter tuning` 代表作。
- 保守判断：
  - 这条线最容易退化成 `analysis / benchmark / tuning note`，很难形成强方法主张。
  - `Inference:` 只要主贡献是“我把参数调得更好”，论文风险就会非常高。

## 10. measurement corruption / delay / dropout / asynchronous measurement

- 已做程度：`A. 明显已被占用`
- 依据：
  - `measurement noise / corrupted data` 这条线已经被 DeePC 核心文献反复处理：`regularized DeePC`、`distributionally robust DeePC`、`IV-DeePC`、`CL-DeePC`，见 [Distributionally Robust Chance Constrained Data-enabled Predictive Control](https://arxiv.org/abs/2006.01702) (2020)、[Data-enabled predictive control with instrumental variables](https://arxiv.org/abs/2209.05210) (2022)、[Closed-loop Data-Enabled Predictive Control...](https://arxiv.org/abs/2402.14374) (2024)。
  - quadrotor DeePC 也明确把噪声测量和正则化作为核心问题，见 [Data-Enabled Predictive Control for Quadcopters](https://www.research-collection.ethz.ch/bitstreams/5972b59c-f24c-4281-90c7-a5670f226763/download)。
- 保守判断：
  - 广义的 measurement corruption 已经是 DeePC 里的成熟主题。
  - 但 `dropout / asynchronous / low-rate vision + high-rate IMU` 这些更细颗粒问题，在我这轮检索里没有被完整占满，所以如果要做，还必须先把问题定义得非常具体，否则还是会滑回旧主线。

## 对本仓库的保守结论

- `DeePC + quadrotor` 这条主干并不空，尤其是 `regularization / noise / hyperparameter tuning / online update` 这些方向，公开代表作已经很多。
- 现在最明显已经被占用、且不适合继续当新主线的方向是：
  - `regularized DeePC`
  - `online data-updated DeePC`
  - `data selection / contextual sampling`
  - `measurement corruption` 的泛化修补
  - `fault / degradation-aware` 的 quadrotor DeePC 变体
- 仍然可以进入候选池、但需要非常谨慎定义问题的方向是：
  - `delay / dropout / asynchronous-measurement-aware DeePC`
  - `safety-filtered / obstacle-aware DeePC`
  - `multi-UAV / formation DeePC`
- 需要特别小心的方向是：
  - `regime-aware / maneuver-conditioned DeePC`
  - 它不是完全空白，但 DeePC 方法线本身已经被 gain-scheduled / regime-varying / local Hankel 的路线占住了。

