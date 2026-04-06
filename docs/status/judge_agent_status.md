# Judge Status

最后更新时间：2026-04-01

## Hard Decisions

1. 公平性问题是否已修复到足以支撑当前方法比较？`no`
2. milestone 3 是否达到？`no`
3. milestone 4 当前是否仍有现实可行性？`no`
4. measurement-aware 方法主线是否继续存活？`no`
5. 下一步应继续：`B. 降级为分析 / 敏感性论文`
6. 单条最强支持证据：冻结矩阵下，`uniform`、`measurement_noise`、`robust_residual_stats` 没有形成跨 `yaw_drift` 和 `anisotropic_noise` 的稳定优势，而 `xyz-only` 又明显过弱，无法支撑“方法成立”。
7. 单条最大 blocker：当前 DeePC 路径仍存在观测流不对称，`Simulation.simulate()` 已采样一次测量，而 `DeePC.compute_input()` 内部又再次采样，导致 controller-to-controller 比较的公平性仍未被关闭；同时即便按现有结果看，残差统计/协方差权重也没有给出多场景稳定赢面。

## Evidence Used

- `deepc/Simulator/simulation.py:82-91` 显示每个采样点先调用 `measure_output(x)` 记录 `y_data`，再调用 controller。
- `deepc/Controllers/deepc.py:290-299` 显示 DeePC 在 `compute_input()` 内部再次调用 `measure_output(x_current)`，并把这次测量用于 `y_ini` / `measurement_residual_d`。
- `deepc/Controllers/lqr.py:16-18` 和 `deepc/Controllers/linear_mpc.py:64-87` 显示这两个 baseline 直接用状态 `x_current`，不走同样的 corrupted measurement 路径，因此不应被写成 measurement-aware 主张的同构公平对照。
- `paper/docs/status/experiment_agent_status.md:7-11` 明确写出 5-seed 冻结矩阵没有任何候选在两个核心坏场景上稳定优于 `uniform regularized DeePC`，且 `xyz-only` 很弱。
- `paper/docs/status/method_agent_status.md:15-18`、`paper/docs/status/method_agent_status.md:74-90` 显示残差统计候选在 smoke test 里没有形成可重复闭环优势，`measurement_noise` 仍是最好但不稳定的非均匀 baseline-like 策略。
- `paper/docs/status/engineering_status.md:40-53` 说明测量层和 regularization 变体已经接入，但这只是工程完成，不是方法成立证据。
- `paper/docs/plans/03_milestones_and_exit_criteria.md:45-64` 定义 milestone 3 需要至少两个核心场景显著改善且多随机种子稳定；当前证据不满足。
- `paper/docs/plans/03_milestones_and_exit_criteria.md:66-82` 定义 milestone 4 需要至少一类关键场景优于 `xyz-only`；现有聚合结果没有给出可防守的重复证据。
- `paper/docs/status/plan_status.md` 和 `paper/docs/status/engineering_status.md` 都已经在正文里把当前项目更接近 sensitivity / analysis paper 作为现实判断。

## Interpretation

- 这轮不能把主线继续写成“measurement-aware method 已经立住”。
- 目前更诚实的表述是：问题已经被造出来，但最小方法没有稳定击穿 `uniform`，更没有击穿 `xyz-only` 这类 pruning 对照。
- 因此下一轮只适合继续做分析型闭环，或者等待真正新的、可复现的权重构造证据。
