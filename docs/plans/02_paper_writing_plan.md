# 论文计划

## 暂定题目

优先题目：

**Measurement-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking**

备选题目：

- **Task-Aligned Regularized DeePC for Quadcopter Position Tracking with Heterogeneous Measurement Uncertainty**
- **Covariance-Aware DeePC for Quadcopter Outer-Loop Tracking under Yaw Drift and Anisotropic Noise**

## 论文定位

这篇文章不是：

- 更快的 DeePC
- 通用 nonlinear DeePC 理论论文
- 实机部署论文
- mode-aware 大系统论文

这篇文章是：

- 四旋翼外环位置跟踪问题上的应用型数据驱动控制论文
- 聚焦 measurement heterogeneity 的 regularized DeePC 变体
- 以系统仿真 benchmark 支撑结论的首篇论文

## 核心问题定义

在 quadcopter outer-loop DeePC 中：

- 位置通道和 `yaw/attitude` 通道的重要性不同
- 测量可信度也不同
- 统一的 regularization / slack 设计会把低质量通道的误差注入 data-consistency fit

因此问题变成：

**如何在不重写 DeePC 框架的前提下，把 measurement quality structure 写进 regularization，使其在 yaw drift、各向异性噪声和参数失配下更稳。**

## 论文贡献

建议只写三点：

1. 指出四旋翼外环 DeePC 中存在 measurement heterogeneity 导致的 consistency-fit 污染问题。
2. 提出基于测量协方差或残差统计的分组加权 regularization / softened consistency 设计。
3. 在纯仿真 benchmark 中系统验证其在 `yaw drift / anisotropic noise / parameter mismatch` 下的鲁棒性提升。

## 文献对话对象

必须正面对话：

- Elokda et al. 2021 quadcopter DeePC
- Coulson et al. 2019 regularized and distributionally robust DeePC
- Huang et al. 2021 quadratic regularization / robust DeePC
- Breschi et al. 2023 regularization impact / uncertainty-aware DDPC
- Chiuso et al. 2025 uncertainty-aware direct DDPC

写法要求：

- 不要声称“首次做四旋翼 DeePC”
- 不要声称“首次发现 noisy orientation channels 有问题”
- 要明确说明你是在锚点论文留下的问题上继续推进

## 方法章节的写法

方法章节重点要落在：

- 为什么只改 tracking cost 不够
- 为什么要改 consistency / slack / regularization
- 为什么权重应与 measurement covariance 或 residual statistics 对应
- 为什么采用 group-wise 而不是完全自由的 per-channel online adaptation

建议结构：

1. 标准 DeePC 回顾
2. 四旋翼外环任务下的问题暴露
3. measurement-aware regularization 形式
4. 权重估计方式
5. 与 `uniform regularization` 的关系

## 实验章节要回答的问题

主实验回答：

- `uniform regularization` 是否在异方差输出下系统性变差
- 你的方法是否比原始 DeePC 更稳
- 你的方法是否优于简单 `xyz-only` pruning

消融回答：

- 收益来自哪里
- 对权重误差是否敏感
- 输出集合不同会不会改变结论

## 图表规划

至少准备：

- 方法框图 1 张
- 系统结构图 1 张
- 主结果表 1 张
- 典型轨迹图 2 到 3 张
- 消融图 2 张
- 协方差错配敏感性图 1 张
- 失败案例图 1 张

## 审稿人可能的攻击点

### 攻击点 1

“这不就是调权重吗？”

防守：

- 强调改的是 DeePC 的 consistency/slack regularization，而不是单纯 tracking cost
- 加入 `manual group-wise` baseline

### 攻击点 2

“这不如直接删掉 noisy output”

防守：

- 加入 `xyz-only DeePC` baseline
- 证明在 `xyzψ` 条件下结构化加权优于直接 pruning

### 攻击点 3

“为什么不用 estimator + MPC”

防守：

- 明确定位是 lightweight DeePC pipeline 的 practical robustness 设计
- 不 claim 替代 estimator-based MPC

### 攻击点 4

“没有硬件，证据不够”

防守：

- 提供多轨迹、多扰动、多种子、多统计结果
- 把仿真实验做成系统 benchmark，而不是演示视频

## 论文停机条件

- 如果 `covariance-aware` 不能稳定优于 `uniform regularization`，论文主 claim 不成立。
- 如果它稳定输给 `xyz-only DeePC`，方法价值要重估。
- 如果只能靠极端参数才能赢，说明方法不稳，论文需要收缩。
