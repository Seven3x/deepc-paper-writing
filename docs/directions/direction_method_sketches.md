# Direction Method Sketches

最后更新时间：2026-04-01

这份草图只保留 3 条我认为还值得试的最小主线。它们的共同点不是“更激进”，而是**改变 DeePC 看到的数据结构**，而不是继续在 `measurement-aware regularization` 上调 `lambda` 或换权重公式。

我刻意没有把 `online data update`、`data selection`、`hyperparameter tuning` 和新的 `measurement-aware weighting tweak` 放进这三条里，因为它们要么已经被 2025 的 DeePC 近邻工作占掉，要么很容易再次滑回“又一个 tweak”。

## 1) Delay / Dropout / Asynchronous-Measurement-Aware DeePC

- 方法核心：把 `y_ini` 从“假设每一步都齐全的历史窗口”改成“带时间对齐和可缺失掩码的历史窗口”；DeePC 仍然解同类 QP，但输入到约束里的历史观测先经过固定延迟对齐、丢包掩码和多速率对齐。
- 最接近已做工作：`CL-DeePC` [arXiv 2024](https://arxiv.org/abs/2402.14374) 和更早的闭环 / 鲁棒 DeePC 主线；但我没有查到一个明确的 `quadrotor + DeePC + asynchronous measurement` 代表作。
- 新意风险：中低。一般 DeePC 的闭环/噪声问题已经很拥挤，但“异步观测链 + 四旋翼轻仿真”这一组合还没被明显占满。
- 实现风险：中低。当前仓库只需要在 `deepc/Simulator/simulation.py`、`deepc/Controllers/deepc.py` 和 `deepc/run_experiment.py` 里加延迟/丢包参数和对齐逻辑；不需要新平台。
- 论文潜力：中高。如果结果成立，它比继续修正测量权重更像一个独立问题，因为它处理的是**观测可用性**而不是**观测权重**。
- 改动位置：
  - `[deepc/Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)`：给 `y_ini` 增加 mask / alignment 参数，约束里只对可观测分量施加等式。
  - `[deepc/Simulator/simulation.py](/home/roxy/deepc-paper/deepc/Simulator/simulation.py)`：注入固定延迟、随机 dropout 和异步采样。
  - `[deepc/compare_measurement_scenarios.py](/home/roxy/deepc-paper/deepc/compare_measurement_scenarios.py)` / `[deepc/run_experiment.py](/home/roxy/deepc-paper/deepc/run_experiment.py)`：增加 delay / dropout / multi-rate 场景参数。
- 为什么不是纯调参：主变化是“哪些历史点进入 DeePC 约束”以及“哪些点缺失”，不是把同一约束换个权重继续压。
- 为什么不是纯平台工程：不需要换仿真器，不需要 PX4 / Gazebo / 实机，只是在现有轻量仿真里改观测流。
- 最可能失败的原因：延迟太小会和 baseline 看起来差不多；延迟或 dropout 稍大又可能让问题变成频繁 infeasible。
- 如果失败，最容易退化成什么类型论文：`analysis / benchmark`，即“DeePC 对异步观测链的敏感性研究”。
- 后续扩展可能性：如果固定延迟有效，再考虑多速率融合或轻量状态估计，但不要一开始就上。

## 2) Fault / Degradation-Aware DeePC

- 方法核心：把“系统健康状态”显式当成一个粗粒度模式变量，只做最小的两态版本，例如 `nominal` vs `degraded`；每个状态对应独立 Hankel bank，运行时按一个简单 health gate 选 bank，而不是继续给统一 regularization 加权。
- 最接近已做工作：`degradation-aware DeePC of energy hubs` [arXiv 2023](https://arxiv.org/abs/2307.01543)，以及四旋翼故障容错控制的相邻工作，如 rotor-failure FTC [arXiv 2020](https://arxiv.org/abs/2002.07837) 和学习式 PFTC [arXiv 2025](https://arxiv.org/abs/2503.02649)。但我没有查到一个明确公开的 `quadrotor + DeePC + fault/degradation-aware` 代表作。
- 新意风险：中低。DeePC 的“退化/健康”概念在别的系统里出现过，但搬到四旋翼轻仿真仍然有空档。
- 实现风险：中低。最小版本只需要在 `[deepc/quadcopter.py](/home/roxy/deepc-paper/deepc/quadcopter.py)` 里加一个单通道退化注入，再在 `[deepc/Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)` 里维护两套 Hankel 数据。
- 论文潜力：中等。它比 measurement-aware 更像一个独立问题，但如果 fault 信号太弱，结论可能只剩下“在某些退化模式下更稳”。
- 改动位置：
  - `[deepc/quadcopter.py](/home/roxy/deepc-paper/deepc/quadcopter.py)`：加入最小的 actuator-efficiency drop 或单传感器偏置/漂移开关。
  - `[deepc/Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)`：增加两 bank Hankel 选择逻辑，先别做连续学习。
  - `[deepc/compare_deepc_regularization.py](/home/roxy/deepc-paper/deepc/compare_deepc_regularization.py)`：新增 `nominal / degraded` 对比矩阵。
- 为什么不是纯调参：bank 切换改变了可用数据集和局部线性化支持集，不是单纯改 slack 权重。
- 为什么不是纯平台工程：只需要故障注入和 bank 管理，不需要额外规划器或通信栈。
- 最可能失败的原因：fault detector 不稳定，或者 bank 分裂后数据太少，结果反而不如 global DeePC。
- 如果失败，最容易退化成什么类型论文：`benchmark / negative result`，即“DeePC 在轻量故障场景下能否分 bank 的实证比较”。
- 后续扩展可能性：若两态 bank 有信号，再补一个极小的 fault classifier；不要一开始就做多故障、多级别、端到端联合学习。

## 3) Regime / Maneuver-Conditioned DeePC

- 方法核心：按机动工况分区构造 local Hankel bank，例如 `hover-like / translation-like / aggressive-turn-like`，运行时按一个可测 scheduling variable 选择最接近的 bank；最小版本不做复杂插值，只做硬选择。
- 最接近已做工作：`GS-DeePC` [arXiv 2025](https://arxiv.org/abs/2509.26334)、`Select-DeePC` [arXiv 2025](https://arxiv.org/abs/2503.18845) 和 `Contextual Sampling` [arXiv 2025](https://arxiv.org/abs/2503.23890)。这条线在一般 DeePC 里已经被明显推进，所以我把它放到三条里但不排第一。
- 新意风险：中高。方向本身合理，但一般 DeePC 的 regime / selection / scheduling 近邻工作太新，撞题风险比前两条更高。
- 实现风险：低到中。现有仓库已经有 `step`、`figure8` 和不同扰动场景，最小版只需在 `[deepc/Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)` 增加 bank selector，并在 `[deepc/trajectory_generator.py](/home/roxy/deepc-paper/deepc/trajectory_generator.py)` 或实验脚本里标记工况。
- 论文潜力：中等偏上，但前提是你能把“quadrotor 的 maneuver-dependent data geometry”讲清楚；否则很容易被读成另一版 data selection。
- 改动位置：
  - `[deepc/Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)`：支持多个局部 Hankel 矩阵和一个简单 scheduling 变量。
  - `[deepc/compare_deepc_regularization.py](/home/roxy/deepc-paper/deepc/compare_deepc_regularization.py)`：把 bank 选择和现有轨迹矩阵接起来。
  - `[deepc/trajectory_generator.py](/home/roxy/deepc-paper/deepc/trajectory_generator.py)`：如果需要，补一个更明确的 aggressive turn / curvature 轨迹。
- 为什么不是纯调参：变化的是局部数据支撑集和 operating-region 归属，不是把统一 DeePC 的惩罚项再调一遍。
- 为什么不是纯平台工程：不需要新硬件；只是把已有轻量轨迹和数据分区规则组织起来。
- 最可能失败的原因：分区太粗会 chattering，分区太细会数据稀疏；另外这条线的 general DeePC 近邻工作太近，方法主张容易显得弱。
- 如果失败，最容易退化成什么类型论文：`analysis / benchmark`，即“不同机动分区对 DeePC 的影响”。
- 后续扩展可能性：如果硬分区有信号，再考虑非常小的邻域插值；不要一开始就把它做成复杂 switching system。

## 保守排序

1. `Delay / Dropout / Asynchronous-Measurement-Aware DeePC`
2. `Fault / Degradation-Aware DeePC`
3. `Regime / Maneuver-Conditioned DeePC`

这个排序只表示在**当前仓库约束**下的可做性和撞题风险，不表示理论上谁更“高级”。
