# 总计划状态

最后更新时间：2026-04-01 15:02

## 总体目标

主线保持为：

`Measurement-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking`

## 阶段状态

### 阶段 1：工程与 baseline

- 状态：进行中，`LQR / linear MPC / original regularized DeePC` 都已接入并跑过最小实验
- 目标：把 `deepc/` 整理成可重复实验入口，并跑通 `LQR / linear MPC / original regularized DeePC`
- 当前结论：
  - `LQR`、`linear MPC` 与 `original regularized DeePC` 已实际跑通最小实验
  - 新的非交互入口已建立，结果会落到 `deepc/Results/<run_name>/`
  - `regularized DeePC` 已通过第一轮短程 OFAT 烟测
  - `regularized DeePC` 已通过第二轮 `step + figure8`、三随机种子验证
  - 当前短程最优候选为 `T_ini=4, N=10, lambda_y=300, lambda_g=3, lqr_noise=0.02`
  - 当前长时段低噪声候选为 `T_ini=8, N=10, lambda_y=1000, lambda_g=10, lqr_noise=0.02`
  - `linear MPC` 已完成 horizon 烟测，当前默认候选可取 `N=12`

### 阶段 2：测量模型层

- 状态：未开始
- 目标：补 `yaw bias / yaw drift / anisotropic noise`

### 阶段 3：measurement-aware regularization

- 状态：未开始
- 目标：在现有 regularized DeePC 上加入 measurement-aware / covariance-aware 权重

### 阶段 4：论文写作

- 状态：未开始
- 进入条件：主实验最小闭环成立，且 baseline 对比可重复

## 当前阻塞

- `regularized DeePC` 的高噪声 `step` 长时段仍不稳定
- `linear MPC` 的长时段 `step` 性能仍弱于理想对照
- 求解速度仍高于 `LQR`，但已能做小规模系统烟测

## 最近重要输出

- 已确认计划执行顺序应先完成工程与 baseline，而不是先写论文
- 已建立状态文档，后续会在这里持续记录关键结论、命令和实验结果
- 已完成两次最小烟测：
  - `LQR`：`deepc/Results/lqr_step_20260401_104619_smoke`
  - `regularized DeePC`：`deepc/Results/deepc_step_20260401_105001_smoke`
- 已完成第一轮收敛烟测：
  - `deepc/Results/deepc_smoke_20260401_113538_round1`
- 已完成第二轮稳定性验证：
  - `step + figure8`
  - `seed = 42 / 7 / 123`
- 已完成第三轮长时段验证：
  - `figure8` 较稳
  - `step` 需要更强正则和更长初始化窗口
- 已完成 `linear MPC` 最小烟测：
  - `deepc/Results/mpc_step_20260401_145247_smoke`
  - `deepc/Results/mpc_figure8_20260401_145304_smoke`
- 已完成 `linear MPC` horizon 烟测与长时段检查：
  - 推荐短程默认 `N=12`
  - 长时段 `figure8` 可用
  - 长时段 `step` 较弱
- 已完成统一 baseline 对比：
  - `deepc/Results/baseline_compare_20260401_150138_smoke`
  - `5 / 6` nominal 运行通过稳定门槛
- 当前阶段性判断：
  - 工程改造方向是对的
  - 论文主线暂时不要往前推
  - 短程稳定 baseline 基本成立
  - 长时段低噪声 baseline 也已出现
  - baseline smoke test 可以视为已完成
  - 现在应按计划进入测量模型层
