# Parent Consolidated Status

最后更新时间：2026-04-01

## Current Real Status

- 老计划的主线是：`Measurement-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking`
- 当前工程底座已经够用：
  - 非交互实验入口已存在
  - baseline 和测量层都已接入
- 问题本身也已经被明确构造出来：
  - `yaw_drift`
  - `anisotropic_noise`
- 但最关键的现实是：
  - 当前 measurement-aware 方法主线还没有被结果支持

## What The Old Plan Implied

- 旧计划默认希望沿下面路径推进：
  - 先有稳定 baseline
  - 再证明 `uniform DeePC` 在坏测量场景下退化
  - 然后用 measurement-aware regularization 在至少两个核心场景上稳定赢回去
  - 再证明这不是简单 `xyz-only` pruning
- milestone 3 的门槛很明确：
  - 至少两个核心场景改善
  - 多随机种子趋势稳定
- milestone 4 进一步要求：
  - 新方法至少在某个关键场景里优于 `xyz-only`

## What The Latest Evidence Actually Supports

- fairness agent 结论：
  - `LQR` / `linear MPC` 不走同一 measurement corruption path，所以不能作为 measurement-aware claim 的公平对照
  - `xyz-only` 必须保留，但它只是 pruning control，不是强 baseline
- method agent 结论：
  - 新增的 `residual_variance`
  - `residual_bias_variance`
  - `robust_residual_stats`
  - 在 smoke test 里都没有形成 defensible win
- experiment evidence：
  - 冻结矩阵的 `5-seed` 聚合已经完成
  - `measurement_noise` 没有稳定优于 `uniform`
  - `robust_residual_stats` 只在 `step + yaw_drift` 的位置项上有局部改善，但 yaw 变差
  - 所有 `anisotropic_noise` 组的 `success_rate` 都是 `0.0`
- judge agent 结论：
  - milestone 3: `no`
  - milestone 4: `no`
  - 当前更适合降级为 sensitivity / analysis paper

## Mismatches That Must Be Surfaced

- 本仓库没有找到用户要求里提到的 `AGENTS.md`，所以本轮是按 `README` 和 `paper/docs/*` 执行的。
- method agent 推荐主实验候选是：
  - `measurement_noise`
  - `residual_variance`
- 但 multi-seed runner 实际跑的是：
  - `measurement_noise`
  - `robust_residual_stats`
- 这意味着：
  - `residual_variance` 还没有拿到 multi-seed 聚合
  - 但这不会逆转当前主判断，因为 `measurement_noise` 已经没有稳定赢面，而 `residual_variance` 的 smoke 也没有更强信号

## What Is Now Frozen

- trajectories:
  - `step`
  - `figure8`
- scenarios:
  - `yaw_drift`
  - `anisotropic_noise`
- controller family:
  - DeePC only
- valid comparison set for the claim:
  - `uniform`
  - `measurement_noise`
  - one residual-statistics candidate
  - `xyz-only`

## Exact Next 3 Actions For The Next Codex Run

1. 不再扩展新方法，只补一个最小验证：
   - 用当前 frozen matrix 给 `residual_variance` 跑一次和本轮同样的 multi-seed 聚合
   - 目的不是救主线，而是把“推荐候选未聚合验证”的缺口补齐
2. 修 fairness debt：
   - 明确隔离 nominal engineering baseline 与 measurement-aware claim baseline
   - 若继续保留 `LQR/MPC`，只允许它们出现在 nominal 或工程 sanity-check 段落
3. 开始改写项目定位：
   - 把主叙事从“measurement-aware method 已成立”改成
   - “measurement heterogeneity / output structure / regularization sensitivity analysis in quadcopter DeePC”

## Is The Paper Mainline Still Alive?

- 作为“measurement-aware method paper mainline”：
  - `no`
- 作为“sensitivity / analysis paper with measurement-aware variants as one branch”：
  - `yes`

## Bottom Line

- 工程没有白做，问题也不是假的。
- 真正失败的是：
  - 当前方法版本没有把这个问题转化成一个可防守的新方法主张。
- 所以下一轮不该继续乐观外推，而该收缩为：
  - 分析型论文定位
  - 或者一次非常小的补验证，用来彻底关掉 residual-statistics 分支。
