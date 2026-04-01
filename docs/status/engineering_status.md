# 工程执行状态

最后更新时间：2026-04-01 10:55

## 目标

按 [plan_index.md](/home/roxy/deepc-paper/paper/docs/plans/plan_index.md) 先跑通工程与 baseline，优先完成：

- 整理 `deepc/` 为可重复实验入口
- 复现 `LQR`
- 复现 `original regularized DeePC`
- 确认 `linear MPC` 当前是否已有实现

## 当前状态

- 状态：已跑通最小工程链路，继续收敛 baseline
- 主执行目录：`/home/roxy/deepc-paper`
- 代码入口：`deepc/main.py`
- 已发现 conda：`/home/roxy/miniconda3/bin/conda`
- 已发现环境：`base`、`delivery`、`game`、`milp`、`ros_noetic`
- 已新建环境：`deepc-paper`
- 本轮实际使用环境：`delivery`

## 已确认事实

- 当前仓库已有控制器实现：
  - `deepc/Controllers/lqr.py`
  - `deepc/Controllers/deepc.py`
- 当前仓库未看到现成 `linear MPC` 控制器文件
- 当前主入口 `deepc/main.py` 为交互式脚本，不适合批量复现实验
- 当前代码内部原先使用了硬编码目录 `DeePC_Quadcopter/...`，已改为基于 `deepc/paths.py` 的相对输出目录
- 已新增非交互实验入口：`deepc/run_experiment.py`
- 已新增 `LQR` 跟踪包装器：`deepc/Controllers/lqr_tracking.py`
- 已修复 `trajectory_generator.py` 中参考轨迹耗尽时的空数组边界问题

## 风险与缺口

- `linear MPC` baseline 目前仍是实现缺口
- `regularized DeePC` 已跑通，但当前参数下在后段明显发散
- `cvxpy` 给出 DPP 效率警告，当前滚动优化速度偏慢
- `cvxpy` 给出 `Solution may be inaccurate` 告警，后续要检查求解器与缩放

## 最近动作

- 已阅读 `paper/docs/plans/plan_index.md`
- 已完成仓库入口初步梳理
- 已定位 conda 可执行文件与环境列表
- 已把交互式脚本补成可重复实验入口
- 已补齐 `delivery` 环境所缺依赖：`control`、`cvxpy`、`h5py`、`sympy`
- 已完成 `LQR` 最小烟测
- 已完成 `regularized DeePC` 最小烟测

## 本轮执行命令

环境检查：

```bash
"$HOME/miniconda3/bin/conda" env list
MPLBACKEND=Agg "$HOME/miniconda3/bin/conda" run -n delivery python -c "import numpy,scipy,matplotlib,cvxpy,control,h5py; print('delivery-deps-ok')"
```

最小 `LQR`：

```bash
MPLBACKEND=Agg "$HOME/miniconda3/bin/conda" run -n delivery python deepc/run_experiment.py \
  --controller lqr --trajectory step --reference-duration 6 --sampling-time 0.1 \
  --dt 0.001 --seed 42 --tag smoke --save-hdf5
```

最小 `regularized DeePC`：

```bash
MPLBACKEND=Agg "$HOME/miniconda3/bin/conda" run -n delivery python deepc/run_experiment.py \
  --controller deepc --trajectory step --reference-duration 6 --sampling-time 0.1 \
  --dt 0.001 --seed 42 --lqr-noise 0.1 --tag smoke --save-hdf5
```

## 关键结果

### `LQR` 烟测

- 结果目录：`deepc/Results/lqr_step_20260401_104619_smoke`
- 指标：
  - `rmse_all_outputs = 0.2938`
  - `rmse_position = 0.4140`
  - `rmse_yaw = 0.00178`
  - `final_position_error_norm = 0.1294`
- 结论：最小 `LQR` 跟踪可运行且末端误差不大，可作为工程 sanity check

### `regularized DeePC` 烟测

- 结果目录：`deepc/Results/deepc_step_20260401_105001_smoke`
- 关键过程：
  - 在 `33.8s` 左右构造 Hankel
  - 打印 `Data is persistently exciting of order 43!`
- 指标：
  - `rmse_all_outputs = 0.3814`
  - `rmse_position = 0.3489`
  - `rmse_yaw = 0.6593`
  - `final_position_error_norm = 4.2027`
- 结论：工程上已经跑通 `original regularized DeePC`，但这组最小设置在后段明显失稳，暂时不能当成稳定 baseline

## 下一步

- 把 `regularized DeePC` 烟测收缩成更稳定的 baseline 设定
  - 优先检查 `T_ini / N / lambda_y / lambda_g / 初始噪声强度`
- 给 `run_experiment.py` 增加可调超参数入口，避免手改源码
- 明确是否现在补 `linear MPC`
  - 如果按计划严格推进，下一步应该补这个 baseline
