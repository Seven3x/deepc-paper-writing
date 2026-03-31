# Engineering Map

## 目标

把 [DeePC_Quadcopter](/home/roxy/deepc-paper/deepc) 从单次交互式仿真原型，整理成适合当前论文主线的实验底座。

当前论文主线是：

**Measurement-Aware / Covariance-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking**

## 代码目录总览

- [main.py](/home/roxy/deepc-paper/deepc/main.py)
  当前主入口。负责交互式选择仿真、动画和结果保存。
- [Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)
  DeePC 控制器核心。当前已有 `y_ini`、`sigma_y`、softened consistency 和统一 regularization。
- [Controllers/lqr.py](/home/roxy/deepc-paper/deepc/Controllers/lqr.py)
  初始控制器和基线之一。
- [quadcopter.py](/home/roxy/deepc-paper/deepc/quadcopter.py)
  四旋翼模型、输出定义、约束与成本矩阵。
- [trajectory_generator.py](/home/roxy/deepc-paper/deepc/trajectory_generator.py)
  参考轨迹生成。
- [Simulator/simulation.py](/home/roxy/deepc-paper/deepc/Simulator/simulation.py)
  仿真循环。
- [hdf5_reader.py](/home/roxy/deepc-paper/deepc/hdf5_reader.py)
  当前结果保存与读取。

## 与论文最相关的已知实现事实

### `main.py`

当前问题：

- 入口是交互式 CLI
- 不适合批量实验
- 配置分散在脚本里

建议改造：

- 提供统一实验配置入口
- 支持非交互式单次运行
- 支持批量运行脚本调用

### `Controllers/deepc.py`

当前已有：

- `y_ini`
- `u_ini`
- `Y_p g = y_ini + sigma_y`
- `lambda_y * ||sigma_y||`
- `lambda_g * ||g - g_r||`

这是当前论文最关键的改造点。

建议改造：

- 保留原始 `uniform regularization`
- 增加 `manual group-wise regularization`
- 增加 `covariance-aware regularization`
- 支持 `xyz` 和 `psi` 分组

### `quadcopter.py`

当前已有：

- 输出索引集中定义
- `C` 矩阵集中生成
- `Q` 和 `R` 在同一处定义

这是测量层和输出集合改造的主位置。

建议改造：

- 支持 `full output`
- 支持 `xyzψ`
- 支持 `xyz`
- 加入测量层
  - anisotropic measurement noise
  - yaw bias
  - yaw drift
- 记录或导出 measurement covariance / residual statistics

## 建议新增的工程内容

建议新增：

- `scripts/`
  - 单次实验脚本
  - 批量实验脚本
  - 汇总结果脚本
- `configs/`
  - baseline 配置
  - measurement mismatch 配置
  - covariance-aware 配置
- `results/` 或统一实验输出目录
  - 指标表
  - 轨迹
  - 配置快照

## 文件级改造优先级

1. [main.py](/home/roxy/deepc-paper/deepc/main.py)
   先把交互式入口改成可复现实验入口。
2. [quadcopter.py](/home/roxy/deepc-paper/deepc/quadcopter.py)
   先补测量层和输出配置。
3. [Controllers/deepc.py](/home/roxy/deepc-paper/deepc/Controllers/deepc.py)
   再加 covariance-aware regularization。
4. 新增脚本目录
   最后补批量实验。

## 当前最推荐的工程推进顺序

1. 先复现 baseline：
   - LQR
   - linear MPC
   - original regularized DeePC
2. 再显式构造问题：
   - yaw drift
   - anisotropic noise
   - parameter mismatch
3. 再上最小方法：
   - `xyzψ + covariance-aware regularization`
4. 最后做系统 benchmark：
   - 多轨迹
   - 多扰动
   - 多随机种子
