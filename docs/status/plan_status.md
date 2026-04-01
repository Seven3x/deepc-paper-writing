# 总计划状态

最后更新时间：2026-04-01 10:55

## 总体目标

主线保持为：

`Measurement-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking`

## 阶段状态

### 阶段 1：工程与 baseline

- 状态：进行中，最小工程链路已打通
- 目标：把 `deepc/` 整理成可重复实验入口，并跑通 `LQR / linear MPC / original regularized DeePC`
- 当前结论：
  - `LQR` 与 `original regularized DeePC` 已实际跑通最小实验
  - `linear MPC` 还未在代码中发现，仍是缺口
  - 新的非交互入口已建立，结果会落到 `deepc/Results/<run_name>/`
  - 当前 `regularized DeePC` 还没有稳定成可用 baseline

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

- `linear MPC` 基线存在实现缺口
- `regularized DeePC` 当前烟测会在后段发散
- 求解速度偏慢，调参迭代成本较高

## 最近重要输出

- 已确认计划执行顺序应先完成工程与 baseline，而不是先写论文
- 已建立状态文档，后续会在这里持续记录关键结论、命令和实验结果
- 已完成两次最小烟测：
  - `LQR`：`deepc/Results/lqr_step_20260401_104619_smoke`
  - `regularized DeePC`：`deepc/Results/deepc_step_20260401_105001_smoke`
- 当前阶段性判断：
  - 工程改造方向是对的
  - 论文主线暂时不要往前推
  - 先把稳定 baseline 收出来再进入测量模型层
