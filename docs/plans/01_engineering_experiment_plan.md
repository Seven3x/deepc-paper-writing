# 工程与实验计划

## 目标

把 `DeePC_Quadcopter` 从“能跑单次仿真”的研究原型，整理成“能做系统 benchmark”的实验底座，并在其上实现第一版 `measurement-aware regularized DeePC`。

## 主线任务

### 阶段 A：平台整理

目标：

- 能统一切换控制器、轨迹、输出集合、噪声模型、扰动模型、随机种子。

任务：

- 梳理入口文件：`main.py`
- 梳理控制器实现：`Controllers/deepc.py`
- 梳理系统输出定义：`quadcopter.py`
- 增加统一配置入口
- 增加结果保存格式，例如 `csv/json/npy`
- 增加批量实验脚本

完成标准：

- 可以通过一组参数启动单次实验
- 可以批量跑多个种子和多个场景
- 每次实验都能落盘指标与轨迹

### 阶段 B：基线复现

目标：

- 先确认原仓库在基础场景下稳定，再引入新方法。

必做基线：

- `LQR`
- `linear MPC`
- `original regularized DeePC`

基础场景：

- `hover`
- `step`
- `box`
- `figure-8`

完成标准：

- 每个基线都能稳定完成基础场景
- 指标可重复生成
- 主要可视化图能自动导出

### 阶段 C：测量模型层

目标：

- 显式引入“测量通道质量异构”这个论文核心问题。

必做改造：

- `yaw bias`
- `yaw drift`
- 各向异性测量噪声
- 参数失配：
  - `mass variation`
  - `thrust coefficient mismatch`
- 外扰：
  - `constant wind`

建议实现方式：

- 在测量层而不是控制器层注入噪声与偏差
- 区分位置通道和 `psi` 通道的噪声强度
- 同时支持已知协方差和故意错配协方差

完成标准：

- 所有失配都可通过参数切换
- 不同失配场景可独立开关

### 阶段 D：第一版方法实现

目标：

- 做出最小但可解释的 `measurement-aware DeePC`。

建议第一版：

- 输出集合优先做 `xyzψ`
- 对 `Y_p g - y_ini` 的 softened consistency / slack 做分组加权
- 权重来源先用离线协方差或残差统计

版本拆分：

- `uniform regularization`
- `manual group-wise regularization`
- `covariance-aware regularization`

暂时不做：

- 在线自适应权重
- mode-aware bank switching
- fixed-budget data selection

完成标准：

- 新方法可以在不改整体框架的前提下运行
- 与原始 DeePC 只差一层 regularization 结构

### 阶段 E：主实验

目标：

- 验证主假设是否成立。

主问题：

- 在 `yaw drift / anisotropic noise / parameter mismatch` 下，`covariance-aware regularization` 是否优于 `uniform regularization`。

主指标：

- 位置 `RMSE`
- 最大位置误差
- 成功率
- 控制输入幅值
- 控制输入平滑性

完成标准：

- 至少在两个失配场景下稳定优于 `uniform`
- 不低于 `xyz-only DeePC`

### 阶段 F：消融与敏感性分析

必做问题：

- 收益来自分组 regularization，还是只是删掉 `psi`
- 手调权重和协方差驱动权重差距多大
- 协方差估计错了以后会不会崩

必做对比：

- `full output` vs `xyzψ` vs `xyz`
- `uniform` vs `manual group-wise` vs `covariance-aware`
- `covariance correct` vs `covariance misspecified`

## 工程优先级

先做：

1. 平台整理
2. 基线复现
3. 测量模型层
4. 第一版方法实现

后做：

1. 大规模消融
2. 更复杂的自适应权重
3. 额外花哨可视化

## 当前建议的文件级落点

- `DeePC_Quadcopter/main.py`
  - 实验入口与配置整理
- `DeePC_Quadcopter/Controllers/deepc.py`
  - regularization / slack 改造主位置
- `DeePC_Quadcopter/quadcopter.py`
  - 输出定义、测量层、噪声与偏差注入
- 新增实验脚本
  - 批量运行
  - 指标汇总
  - 结果落盘

## 工程停机条件

- 如果新方法连 `xyz-only DeePC` 都打不过，主方法需要收缩。
- 如果收益只存在于单一轨迹或单一噪声设置，说明主 claim 太弱。
- 如果主要收益来自重新调 `Q` 而不是 regularization 结构，需要重写方法定位。
