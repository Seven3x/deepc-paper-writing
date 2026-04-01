# 工程执行状态

最后更新时间：2026-04-01 17:20

## 目标

按 [plan_index.md](/home/roxy/deepc-paper/paper/docs/plans/plan_index.md) 先跑通工程与 baseline，优先完成：

- 整理 `deepc/` 为可重复实验入口
- 复现 `LQR`
- 复现 `original regularized DeePC`
- 确认 `linear MPC` 当前是否已有实现

## 当前状态

- 状态：baseline 三件套均已接入工程入口
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
- `linear MPC` 已新增实现：`deepc/Controllers/linear_mpc.py`
- 当前主入口 `deepc/main.py` 为交互式脚本，不适合批量复现实验
- 当前代码内部原先使用了硬编码目录 `DeePC_Quadcopter/...`，已改为基于 `deepc/paths.py` 的相对输出目录
- 已新增非交互实验入口：`deepc/run_experiment.py`
- 已新增 `LQR` 跟踪包装器：`deepc/Controllers/lqr_tracking.py`
- 已修复 `trajectory_generator.py` 中参考轨迹耗尽时的空数组边界问题
- 已把 `T_ini / N / lambda_y / lambda_g / 初始噪声` 做成 CLI 可调
- 已新增 OFAT 烟测脚本：`deepc/sweep_deepc_smoke.py`
- 已新增 `deepc/.gitignore`，忽略缓存和运行产物目录
- 已修复 `figure8` 参考生成中的除零 warning
- 已将 `linear MPC` 接入 `deepc/run_experiment.py`
- 已在 `quadcopter.py` 接入测量层失配注入：
  - `yaw bias`
  - `yaw drift`
  - 各向异性测量噪声
- 已在 `run_experiment.py` 暴露测量层 CLI 参数
- 已新增测量场景对比脚本：`deepc/compare_measurement_scenarios.py`
- 已在 `Controllers/deepc.py` 接入第一版非均匀 `sigma_y` 权重
- 已新增正则化对比脚本：`deepc/compare_deepc_regularization.py`
- 已接入基于测量噪声统计的自动权重模式：`measurement_noise`
- 已接入基于测量残差统计的自动权重模式：`residual_stats`
- 已接入可切换输出集合：`xyzpsi / xyz`
- 已接入 `RandomExcitationController`，用于需要额外初始激励的数据收集场景
- 已接入 `block_l2` 结构：对 `roll/pitch / yaw / position` 三块分别惩罚
- 已接入 `drop_yaw_past` 结构：在 past consistency 中直接去掉 `yaw` 行

## 风险与缺口

- `linear MPC` 已完成 horizon 烟测，但长时段 `step` 仍明显偏弱
- 第一轮稳定区间只覆盖了较短 `step` 烟测，还没有跨轨迹验证
- 当前稳定区间已覆盖短程 `step + figure8`
- 长时段验证已完成，`step` 与 `figure8` 的难度明显不同
- `cvxpy` 滚动优化仍偏慢，但切到 `CLARABEL + ignore_dpp=True` 后已可接受
- 长时段验证目前主要是 `seed=42`，还没做多种子复核

## 最近动作

- 已阅读 `paper/docs/plans/plan_index.md`
- 已完成仓库入口初步梳理
- 已定位 conda 可执行文件与环境列表
- 已把交互式脚本补成可重复实验入口
- 已补齐 `delivery` 环境所缺依赖：`control`、`cvxpy`、`h5py`、`sympy`
- 已完成 `LQR` 最小烟测
- 已完成 `regularized DeePC` 最小烟测
- 已完成超参数可调化改造
- 已完成第一轮 OFAT 烟测
- 已完成第二轮多随机种子、多轨迹验证
- 已完成第三轮长时段验证与组合修正
- 已完成 `linear MPC` 最小实现与双轨迹烟测
- 已完成 `linear MPC` horizon 烟测与长时段边界检查
- 已完成统一 baseline 对比
- 已完成测量模型层首轮烟测
- 已完成第一轮 `manual weighting` 方法探索

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

单运行稳定性验证：

```bash
MPLBACKEND=Agg "$HOME/miniconda3/bin/conda" run -n delivery python deepc/run_experiment.py \
  --controller deepc --trajectory step --reference-duration 3 --sampling-time 0.1 \
  --dt 0.001 --seed 42 --lqr-noise 0.02 --deepc-T-ini 4 --deepc-N 8 \
  --deepc-lambda-y 300 --deepc-lambda-g 3 --deepc-solver CLARABEL \
  --quiet --tag tunecheck
```

第一轮 OFAT 烟测：

```bash
MPLBACKEND=Agg "$HOME/miniconda3/bin/conda" run -n delivery python deepc/sweep_deepc_smoke.py \
  --trajectory step --reference-duration 3 --sampling-time 0.1 --dt 0.001 --seed 42 \
  --base-T-ini 4 --base-N 8 --base-lambda-y 300 --base-lambda-g 3 --base-lqr-noise 0.02 \
  --t-ini-values 4,6 --N-values 8,10 --lambda-y-values 300,1000 \
  --lambda-g-values 3,10 --lqr-noise-values 0.02,0.05 --tag round1
```

测量模型层最小矩阵：

```bash
MPLBACKEND=Agg "$HOME/miniconda3/bin/conda" run -n delivery python deepc/compare_measurement_scenarios.py \
  --controllers deepc --trajectories step,figure8 \
  --scenarios nominal,yaw_bias,yaw_drift,anisotropic_noise \
  --reference-duration 6 --sampling-time 0.1 --dt 0.001 \
  --seed 42 --measurement-seed 0 --tag stageC_smoke
```

第一轮正则化对比：

```bash
MPLBACKEND=Agg "$HOME/miniconda3/bin/conda" run -n delivery python deepc/compare_deepc_regularization.py \
  --trajectories step,figure8 --scenarios nominal,yaw_drift,anisotropic_noise \
  --variants uniform,manual_yaw_only \
  --manual-yaw-only-weights 1,1,0.5,1,1,1 \
  --reference-duration 6 --sampling-time 0.1 --dt 0.001 \
  --seed 42 --measurement-seed 0 --tag yaw_only_v2
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

### `linear MPC` 烟测

- 实现文件：`deepc/Controllers/linear_mpc.py`
- 运行入口：`deepc/run_experiment.py --controller mpc`
- `step` 结果目录：`deepc/Results/mpc_step_20260401_145247_smoke`
  - `rmse_position = 0.1683`
  - `rmse_yaw = 0.3542`
  - `final_position_error_norm = 0.00050`
- `figure8` 结果目录：`deepc/Results/mpc_figure8_20260401_145304_smoke`
  - `rmse_position = 0.0577`
  - `rmse_yaw = 0.00213`
  - `final_position_error_norm = 0.0247`
- 结论：
  - 工程上已经跑通 `linear MPC` baseline
  - 当前 `N=10` 的 `step` yaw 指标一般，后续仍值得单独调 horizon

### `linear MPC` horizon 烟测

- 轨迹：`step`, `reference_duration = 3`
  - `N=6`: `rmse_position = 0.2695`, `rmse_yaw = 0.9593`
  - `N=8`: `rmse_position = 0.2371`, `rmse_yaw = 0.7190`
  - `N=10`: `rmse_position = 0.1683`, `rmse_yaw = 0.3542`
  - `N=12`: `rmse_position = 0.1653`, `rmse_yaw = 0.2676`
  - `N=16`: `rmse_position = 0.1648`, `rmse_yaw = 0.2461`
- 轨迹：`figure8`, `reference_duration = 3`
  - `N=6~16` 基本都稳定，指标差异很小
  - `rmse_position` 基本在 `0.0577 ~ 0.0580`
  - `rmse_yaw` 基本在 `0.0019 ~ 0.0021`
- 结论：
  - `step` 对 horizon 明显敏感
  - `N=12` 是当前较合理的折中点
  - `N=16` 比 `N=12` 只带来很小收益，不值得默认使用

### `linear MPC` 长时段边界

- `step, reference_duration = 6, N = 12`
  - 结果目录：`deepc/Results/mpc_step_20260401_145733_mpc_long_N12_fix`
  - `rmse_position = 1.4057`
  - `rmse_yaw = 4.5261`
  - `final_position_error_norm = 5.7319`
  - 结论：能跑完，但控制质量明显不够，不适合作为长时段 `step` 强 baseline
- `figure8, reference_duration = 6, N = 12`
  - 结果目录：`deepc/Results/mpc_figure8_20260401_145802_mpc_long_N12_fix`
  - `rmse_position = 0.0413`
  - `rmse_yaw = 0.00476`
  - `final_position_error_norm = 0.0610`
  - 结论：长时段 `figure8` 仍表现稳定

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

### 收敛后的稳定单运行

- 结果目录：`deepc/Results/deepc_step_20260401_113528_tunecheck`
- 配置：
  - `T_ini = 4`
  - `N = 8`
  - `lambda_y = 300`
  - `lambda_g = 3`
  - `lqr_noise = 0.02`
  - `solver = CLARABEL`
- 指标：
  - `rmse_position = 0.1187`
  - `rmse_yaw = 0.00707`
  - `final_position_error_norm = 0.00157`
- 结论：相比之前默认设置，已经进入稳定工作区

### 第一轮 OFAT 烟测

- 汇总目录：`deepc/Results/deepc_smoke_20260401_113538_round1`
- 总运行数：`6`
- 稳定运行数：`6 / 6`
- 排名最优配置：
  - `T_ini = 4`
  - `N = 10`
  - `lambda_y = 300`
  - `lambda_g = 3`
  - `lqr_noise = 0.02`
  - `rmse_position = 0.1131`
  - `rmse_yaw = 0.00688`
- 敏感性结论：
  - `N: 8 -> 10` 有小幅收益
  - `T_ini: 4 -> 6` 也有小幅收益
  - `lambda_y: 300 -> 1000` 基本无显著收益
  - `lambda_g: 3 -> 10` 略差
  - `lqr_noise: 0.02 -> 0.05` 明显变差，但仍稳定

### 第二轮验证：固定候选参数跨轨迹、跨种子

- 验证配置：
  - `T_ini = 4`
  - `N = 10`
  - `lambda_y = 300`
  - `lambda_g = 3`
  - `lqr_noise = 0.02`
  - `solver = CLARABEL`
- 运行集合：
  - `step`: seeds `42 / 7 / 123`
  - `figure8`: seeds `42 / 7 / 123`
- 结果目录：
  - `deepc/Results/deepc_step_20260401_134401_round2`
  - `deepc/Results/deepc_step_20260401_134408_round2`
  - `deepc/Results/deepc_step_20260401_134414_round2`
  - `deepc/Results/deepc_figure8_20260401_134420_round2`
  - `deepc/Results/deepc_figure8_20260401_134426_round2`
  - `deepc/Results/deepc_figure8_20260401_134432_round2`
- 汇总结论：
  - `6 / 6` 运行全部 `all_finite = true`
  - `step`：
    - `rmse_position` 约 `0.108 ~ 0.113`
    - `rmse_yaw` 约 `0.0069 ~ 0.0090`
    - `final_position_error_norm` 约 `0.00018 ~ 0.00311`
  - `figure8`：
    - `rmse_position` 约 `0.0589 ~ 0.0621`
    - `rmse_yaw` 约 `0.0213 ~ 0.0349`
    - `final_position_error_norm` 约 `0.0165 ~ 0.0241`
- 阶段判断：
  - 这组参数已经可视为“短程稳定 baseline 候选”
  - 下一步不该继续盲目扫超参，应转向更长时段与更高噪声验证

### 第三轮验证：长时段 `reference_duration = 6`

- 旧短程候选：
  - `T_ini = 4`
  - `N = 10`
  - `lambda_y = 300`
  - `lambda_g = 3`
- 长时段结果：
  - `step` 明显失稳
    - `rmse_position = 3.5600`
    - `rmse_yaw = 1.1370`
    - `final_position_error_norm = 35.0760`
  - `figure8` 仍稳定
    - `noise = 0.02` 时 `rmse_position = 0.0552`
    - `noise = 0.05` 时 `rmse_position = 0.0638`

### 第四轮验证：`step-6s` 组合修正

- 单因素 OFAT 汇总目录：
  - `deepc/Results/deepc_smoke_20260401_134625_step6_round1`
- 单因素结论：
  - 单独增大 `T_ini`、`N`、`lambda_y`、`lambda_g` 都无法把 `step-6s` 拉回稳定区
  - 但 `T_ini = 8` 与 `lambda_y = 1000` 显著改善了误差量级

- 组合测试结果：
  - `T_ini=8, N=10, lambda_y=1000, lambda_g=3`
    - 仍不稳定，`final_position_error_norm = 24.56`
  - `T_ini=8, N=10, lambda_y=1000, lambda_g=10`
    - 已进入可用区间
    - 结果目录：`deepc/Results/deepc_step_20260401_134801_step6_combo_b`
    - 指标：
      - `rmse_position = 0.1483`
      - `rmse_yaw = 0.0430`
      - `final_position_error_norm = 0.0607`
      - `max_abs_position_error = 0.6944`
  - `T_ini=8, N=12, lambda_y=1000, lambda_g=3`
    - 仍不稳定
  - `T_ini=6, N=10, lambda_y=1000, lambda_g=3`
    - 仍不稳定

### 组合候选的跨轨迹确认

- 候选参数：
  - `T_ini = 8`
  - `N = 10`
  - `lambda_y = 1000`
  - `lambda_g = 10`
- 确认结果：
  - `step, 6s, noise=0.02`
    - 稳定，可用
  - `step, 6s, noise=0.05`
    - 仍不稳定，`final_position_error_norm = 23.68`
  - `figure8, 6s, noise=0.02`
    - 稳定，`rmse_position = 0.0558`
  - `figure8, 6s, noise=0.05`
    - 稳定，`rmse_position = 0.0680`

## 当前阶段判断

- 若噪声较低，当前可把下面这组参数作为“长时段稳定 baseline 候选”：
  - `T_ini = 8`
  - `N = 10`
  - `lambda_y = 1000`
  - `lambda_g = 10`
- 当前主要剩余难点不是普通跟踪，而是：
  - `step` 轨迹
  - 更高初始激励噪声

### 统一 baseline 对比

- 汇总目录：
  - `deepc/Results/baseline_compare_20260401_150138_smoke`
- 覆盖控制器：
  - `LQR`
  - `linear MPC (N=12)`
  - `regularized DeePC`
- 覆盖轨迹：
  - `step`
  - `figure8`
- 稳定性结果：
  - `5 / 6` 运行通过稳定门槛
  - 唯一未通过的是 `MPC + step`

- `step, 6s`
  - `LQR`
    - `rmse_position = 0.4140`
    - `rmse_yaw = 0.0018`
  - `MPC`
    - `rmse_position = 1.4057`
    - `rmse_yaw = 4.5261`
    - 未通过稳定门槛
  - `DeePC`
    - `rmse_position = 0.1483`
    - `rmse_yaw = 0.0430`

- `figure8, 6s`
  - `LQR`
    - `rmse_position = 0.3208`
    - `final_position_error_norm = 0.9367`
  - `MPC`
    - `rmse_position = 0.0413`
    - `rmse_yaw = 0.0048`
  - `DeePC`
    - `rmse_position = 0.0552`
    - `rmse_yaw = 0.0498`

- 结论：
  - 在 nominal、长时段 `step` 上，当前最强 baseline 是调过参的 `regularized DeePC`
  - 在 `figure8` 上，`MPC` 与 `DeePC` 都明显优于 `LQR`
  - `MPC` 可以保留在对比中，但要明确它不是长时段 `step` 的强 baseline

### 测量模型层首轮烟测

- 汇总目录：
  - `deepc/Results/measurement_compare_20260401_151253_stageC_smoke`
- 当前场景：
  - `nominal`
  - `yaw_bias = 0.2 rad`
  - `yaw_drift = 0.03 rad/s`
  - `anisotropic_noise = [0.005, 0.005, 0.12, 0.01, 0.01, 0.01]`
- 覆盖轨迹：
  - `step`
  - `figure8`
- 覆盖控制器：
  - `regularized DeePC`
- 稳定性结果：
  - `4 / 8` 运行通过稳定门槛
  - `nominal` 与 `yaw_bias` 通过
  - `yaw_drift` 与 `anisotropic_noise` 未通过

- `step, 6s`
  - `nominal`
    - `rmse_position = 0.1483`
    - `rmse_yaw = 0.0430`
  - `yaw_bias`
    - `rmse_position = 0.1400`
    - `rmse_yaw = 0.1854`
  - `yaw_drift`
    - `rmse_position = 0.1355`
    - `rmse_yaw = 0.8191`
    - 未通过稳定门槛
  - `anisotropic_noise`
    - `rmse_position = 1.1439`
    - `rmse_yaw = 2.8229`
    - 未通过稳定门槛

- `figure8, 6s`
  - `nominal`
    - `rmse_position = 0.0552`
    - `rmse_yaw = 0.0498`
  - `yaw_bias`
    - `rmse_position = 0.0556`
    - `rmse_yaw = 0.1813`
  - `yaw_drift`
    - `rmse_position = 1.8284`
    - `final_position_error_norm = 20.4180`
    - 未通过稳定门槛
  - `anisotropic_noise`
    - `rmse_position = 4.5675`
    - `final_position_error_norm = 35.9434`
    - 未通过稳定门槛

- 当前解释：
  - `uniform regularized DeePC` 对静态 `yaw bias` 还有一定容忍度
  - `yaw drift` 已足以显著拉高 `yaw` 误差，并在 `figure8` 上带来位置层面的明显崩坏
  - 各向异性噪声会同时破坏 `step` 与 `figure8` 的稳定性
  - 这已经足够支撑下一步进入 `measurement-aware regularization`
  - 当前 `LQR` 与 `linear MPC` 仍按状态反馈实现，测量层扰动暂时主要直接作用于 `DeePC`

### 第一轮 `manual weighting` 探索

- 核心改动：
  - 在 `sigma_y` 上支持：
    - `uniform`
    - `manual_grouped`
    - `manual_output`
- 汇总目录：
  - `deepc/Results/deepc_reg_compare_20260401_155748_manual_v1`
  - `deepc/Results/deepc_reg_compare_20260401_160021_yaw_only_v1`
  - `deepc/Results/deepc_reg_compare_20260401_160226_yaw_only_v2`

- `manual_grouped`，即整体下调姿态组权重：
  - 配置：`attitude_weight = 0.2`, `position_weight = 1.0`
  - 结论：方向错误
  - 在 nominal 场景就明显破坏闭环，不能继续沿这条线推进

- `manual_yaw_only v1`，即只下调 `psi` 到 `0.2`：
  - 结论：过于激进
  - `step` nominal 还能跑，但 `figure8` nominal 已明显变差
  - 不适合作为默认候选

- `manual_yaw_only v2`，即只下调 `psi` 到 `0.5`：
  - 结果目录：`deepc/Results/deepc_reg_compare_20260401_160226_yaw_only_v2`
  - nominal：
    - `step` 略差于 `uniform`
    - `figure8` 基本持平
  - `yaw_drift`：
    - `step` 的 `final_position_error_norm` 从 `0.1171` 降到 `0.0308`
    - 但 `rmse_yaw` 从 `0.8191` 升到 `0.8468`
    - 仍未过稳定门槛
    - `figure8` 仍劣于 `uniform`
  - `anisotropic_noise`：
    - `step` 只有极小幅改善
    - `figure8` 明显更差

- 当前判断：
  - 第一版手工权重还不能支撑主 claim
  - 简单“下调坏通道权重”不是充分条件
  - 下一步不应继续盲调单个手工权重，而应改成：
    - 基于场景统计的权重标定
    - 或更细的 regularization 结构，而不只是单纯缩放 `sigma_y`

### 第一版 `measurement_noise` 自动权重

- 结果目录：
  - `deepc/Results/deepc_reg_compare_20260401_160622_measurement_noise_v1`
- 权重来源：
  - 直接由测量噪声标准差 `noise_std` 构造
  - nominal 场景自动退化回 `uniform`
- 结果：
  - nominal：
    - `step` 与 `figure8` 和 `uniform` 完全一致
  - `yaw_drift`：
    - 与 `uniform` 完全一致
    - 说明单靠 `noise_std` 处理不了偏置/漂移型失配
  - `anisotropic_noise, step`：
    - `rmse_yaw` 从 `2.8229` 降到 `2.2795`
    - `max_abs_yaw_error` 从 `20.3858` 降到 `14.6244`
    - 位置误差基本不变
    - 但仍未通过稳定门槛
  - `anisotropic_noise, figure8`：
    - 明显更差，`rmse_yaw = 7.6790`

- 当前判断：
  - 这是目前最合理的 measurement-aware 起点，因为：
    - 不破坏 nominal
    - 对异方差 `step` 至少出现了正确方向的改善
  - 但它还远远不够支撑主实验结论
  - 后续若继续，应优先做：
    - 基于残差统计而不是仅 `noise_std`
    - 或对 `step` 与 `figure8` 分开标定

### 第一版 `residual_stats` 自动权重

- 结果目录：
  - `deepc/Results/deepc_reg_compare_20260401_165400_residual_stats_v1`
- 权重来源：
  - 初始数据阶段的测量残差 `y_measured - C x_true`
  - 用每通道残差 RMS 生成 `sigma_y` 权重
  - 当前实现是仿真版残差统计，依赖已知真状态

- 结果：
  - nominal：
    - `step` 与 `figure8` 都和 `uniform` 完全一致
    - 说明它不会破坏 baseline
  - `yaw_drift`：
    - 自动把 `psi` 权重压到 `0.1`
    - 但 `step` 变差：
      - `rmse_yaw: 0.8191 -> 0.9582`
      - `final_position_error_norm: 0.1171 -> 0.3225`
    - `figure8` 也明显更差：
      - `rmse_position: 1.8284 -> 3.2597`
  - `anisotropic_noise, step`：
    - 和 `measurement_noise` 基本同量级
    - `rmse_yaw: 2.8229 -> 2.2874`
    - `max_abs_yaw_error: 20.3858 -> 14.6264`
    - 但位置指标仍没回到稳定区
  - `anisotropic_noise, figure8`：
    - 明显更差，`rmse_yaw = 17.2465`

- 当前判断：
  - `residual_stats` 比手工权重更干净，也不会破坏 nominal
  - 但它并没有优于 `measurement_noise`
  - 对 `yaw_drift` 没形成正收益，对 `figure8` 还会放大失败
  - 这说明“只靠单层 per-output slack reweighting”大概率不够

### `xyz-only DeePC` 基线尝试

- 工程改动：
  - `quadcopter.py` 支持 `--output-set xyz`
  - `run_experiment.py` 支持切换输出集合与 DeePC 初始控制器
  - `Controllers/deepc.py` 修正了数据长度下界，保证 `xyz` 场景下 Hankel 数据长度足以检验 persistency

- 当前最小验证：
  - `step, 3s`
    - 配置：`output_set=xyz, T_ini=4, N=8, lambda_y=300, lambda_g=3`
    - 结果目录：`deepc/Results/deepc_step_20260401_170614_xyz_nom_b`
    - `rmse_position = 3.0553`
    - `final_position_error_norm = 25.6788`
  - `figure8, 3s`
    - 同配置
    - 结果目录：`deepc/Results/deepc_figure8_20260401_170636_xyz_nom_fig8`
    - `rmse_position = 2.1022`
    - `final_position_error_norm = 20.4313`

- 当前判断：
  - `xyz-only DeePC` 已经不是“跑不通”，而是“能跑但 nominal 就明显偏弱”
  - 这意味着在当前工程设置里，简单裁掉 `yaw` 通道并不能自动形成强 baseline
  - 它可以保留为 `pruning` 负对照，但目前不值得继续大规模调参

### 第一版 `block_l2` 结构

- 核心想法：
  - 不再对整个 `sigma_y` 用单一范数
  - 改成对 `roll/pitch`、`yaw`、`position` 三个块分别加范数惩罚
- 最小验证场景：
  - `anisotropic_noise + step, 6s`
  - `yaw` 块系数扫描：
    - `500`
    - `100`
    - `50`
- 结果目录：
  - `deepc/Results/deepc_step_20260401_171424_block_y500`
  - `deepc/Results/deepc_step_20260401_171424_block_y100`
  - `deepc/Results/deepc_step_20260401_171424_block_y50`
- 结果：
  - 三组几乎没有可见差异
  - 代表性结果 `yaw=100`：
    - `rmse_position = 1.1454`
    - `rmse_yaw = 2.8402`
    - `final_position_error_norm = 7.9443`
  - 明显差于 `measurement_noise`：
    - `measurement_noise` 在同场景下 `rmse_yaw = 2.2795`

- 当前判断：
  - 这版 `block_l2` 没形成正收益
  - 至少在当前公式下，“把 `yaw` 单独成块”还不足以改善核心坏场景
  - 继续在这条线上细扫超参，收益预期不高

### 第一版 `drop_yaw_past` 结构

- 核心想法：
  - 不再改 `sigma_y` 代价
  - 直接在 `Y_p g = y_ini` 的 past consistency 里删掉 `yaw` 行
  - 只保留 `roll/pitch/position` 的硬一致性

- 结果目录：
  - `deepc/Results/deepc_step_20260401_172027_drop_yaw_past_aniso`
  - `deepc/Results/deepc_step_20260401_172028_drop_yaw_past_drift`

- 结果：
  - `anisotropic_noise + step`
    - `rmse_yaw = 1.6446`
    - 相比：
      - `uniform = 2.8229`
      - `measurement_noise = 2.2795`
    - 说明在这个场景上它是目前最强的方向
    - 但位置误差没有同步进入稳定区：
      - `rmse_position = 1.3600`
      - `final_position_error_norm = 9.8907`
  - `yaw_drift + step`
    - 明显更差：
      - `rmse_position = 2.5933`
      - `rmse_yaw = 2.5150`
      - `final_position_error_norm = 26.6785`

- 当前判断：
  - 这条结构第一次给出了“在异方差噪声场景上明显优于现有 weighting 方案”的信号
  - 但它对 `yaw_drift` 明显不鲁棒
  - 所以它还不能直接作为统一主方法
  - 更像是：
    - `anisotropic_noise` 的有效结构线索
    - 但不适合直接覆盖 `yaw_drift`

## 下一步

- baseline smoke test 已结束，不再继续扩大 nominal 对比
- 测量模型层已接入并完成首轮烟测
- 下一步建议：
  - `manual weighting / residual_stats / block_l2` 这三条线都不符合预期
  - `drop_yaw_past` 只在 `anisotropic_noise` 上显出强正向信号，但在 `yaw_drift` 上失败
  - 这意味着当前 `plan` 的“统一 measurement-aware 方法”实现路线不符合预期，需要收缩
  - 更合理的收缩方式：
    - 把主问题先收窄到 `anisotropic noise`
    - 或把方法定位改成“场景相关的 channel pruning / consistency gating”
  - 继续围绕 `yaw drift` 与 `anisotropic noise` 做最小主实验
  - 暂时不扩展到 `mass variation / wind`，先把 measurement-aware 主假设做实
