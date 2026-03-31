# 实验矩阵

## 主假设

在四旋翼外环 DeePC 中，若输出通道存在异方差测量噪声和 yaw 相关失配，则：

- `uniform regularization` 会被低可信度通道拖坏
- `covariance-aware / measurement-aware regularization` 能改善闭环跟踪鲁棒性

## 控制器对比

最小集合：

- `LQR`
- `linear MPC`
- `original regularized DeePC`
- `xyz-only DeePC`
- `manual group-wise DeePC`
- `covariance-aware DeePC`

## 输出集合

必做：

- `xyz`
- `xyzψ`

可选：

- `full output`

## 轨迹集合

主实验：

- `hover`
- `step`
- `box`
- `figure-8`

## 失配与扰动

### Measurement mismatch

- `yaw bias`
- `yaw drift`
- 各向异性测量噪声

### Plant mismatch

- 质量变化
- 推力系数扰动

### External disturbance

- 恒定风扰

## 主要实验块

### 实验块 A：Baseline Smoke Test

问题：

- 基线方法是否在 nominal 场景下都能稳定运行。

固定项：

- 基础轨迹
- 无强扰动

变化项：

- 控制器类型

成功标准：

- 所有方法能稳定跑完

### 实验块 B：Measurement Heterogeneity

问题：

- `uniform` 是否会在输出异方差下明显变差。

固定项：

- 控制频率
- 轨迹
- 预测时域

变化项：

- 噪声强度
- `yaw bias / yaw drift`
- regularization 形式

成功标准：

- `covariance-aware` 在核心指标上优于 `uniform`

### 实验块 C：Pruning vs Weighting

问题：

- 你的方法是否优于简单丢弃低质量输出通道。

固定项：

- 同一扰动设置

变化项：

- `xyz`
- `xyzψ`
- `uniform`
- `manual group-wise`
- `covariance-aware`

成功标准：

- 至少一类关键场景中，`covariance-aware xyzψ` 优于 `xyz-only`

### 实验块 D：Parameter Mismatch

问题：

- 方法是否只对 measurement noise 有效，还是对模型失配也更稳。

变化项：

- 质量变化
- 推力系数扰动

成功标准：

- `covariance-aware` 相对 `uniform` 的趋势仍成立

### 实验块 E：Covariance Misspecification

问题：

- 如果协方差估计不准，方法会不会失效。

变化项：

- 协方差估准
- 低估 orientation noise
- 高估 orientation noise

成功标准：

- 方法有一定鲁棒性
- 至少不会明显差于 `manual group-wise`

## 指标

必报：

- 位置 `RMSE`
- 最大位置误差
- 成功率
- 控制输入能量
- 控制输入平滑性

建议补充：

- 多种子均值
- 多种子标准差
- 代表性失败案例

## 运行优先级

先跑：

1. Baseline Smoke Test
2. Measurement Heterogeneity
3. Pruning vs Weighting

后跑：

1. Parameter Mismatch
2. Covariance Misspecification
3. 大规模附加消融

## 结果判定

继续推进的标准：

- 主实验块 B 成立
- 实验块 C 至少部分成立

需要收缩问题的标准：

- 新方法赢不了 `xyz-only`
- 优势只在极少数设置下出现
- 协方差稍微错配就整体崩掉
