# DeePC Quadcopter 首篇论文方向总结（供多 subagent 分析讨论）

## 读前说明

这份文档的作用是：

- 保留最初的三方向发散分析
- 记录为什么曾经优先考虑方向一
- 给后续 agent 一个“全局问题空间”的总览

但请注意：

- **这不是当前最终拍板版本**
- 在后续更深一轮文献调研与工程筛查后，当前最推荐主线已经从“方向二打底 + 方向一主贡献”收缩为：
  - **Measurement-Aware / Covariance-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking**
- 详细的最终结论、方法定义、实验矩阵和工程改造建议，请优先阅读：
  - [recommended_measurement_aware_deepc_direction.md](/home/roxy/deepc-paper/paper/docs/overview/recommended_measurement_aware_deepc_direction.md)

如果你是新 agent，建议阅读顺序：

1. 先看本文件，理解最初的三方向比较和舍弃逻辑
2. 再看 [recommended_measurement_aware_deepc_direction.md](/home/roxy/deepc-paper/paper/docs/overview/recommended_measurement_aware_deepc_direction.md)，执行当前最终方案
3. 若需要开始代码工作，再看 [ENGINEERING_MAP.md](/home/roxy/deepc-paper/paper/docs/engineering/ENGINEERING_MAP.md)

---

## 0. 目标定位

你的目标不是做一个“大而全”的四旋翼 DeePC 系统，而是基于现有轻量仿真底座（如 `DeePC_Quadcopter`）做出一篇**首篇能发出去**的论文。

建议定位：

- 优先级：**先发出去 > 做得很炫**
- 贡献类型：**小而清晰的算法改动 + 充分实验**
- 平台选择：**轻量自写仿真优先，不再把主力投入 Gazebo/PX4 联调**
- 任务收缩：**优先做外环位置跟踪，不要第一篇就端到端接管全系统**

---

## 1. 当前仓库的关键事实

基于 `MartinPetre/DeePC_Quadcopter` 这个仓库，目前它的特点是：

- 是一个**轻量、自写动力学 + DeePC 仿真平台**
- 已经具备：
  - 四旋翼动力学
  - DeePC 控制器
  - LQR 激励数据采集
  - 轨迹跟踪
  - 绘图与动画
- 当前明显问题：
  - **没有 inner loop**
  - DeePC 既管姿态又管位置
  - yaw 漂移会导致机体系 / 惯性系失配
  - 平台更像研究原型，不像标准论文 benchmark 平台

因此它适合做：
- 算法迭代
- 消融实验
- benchmark 自动化

但不适合直接当“论文成品”。

### 补充：后续工程检查得到的更具体信息

在后续针对 `MartinPetre/DeePC_Quadcopter` 的源码检查中，进一步确认了以下事实：

- 仓库是一个**很适合做轻量改造和批量实验的研究原型**
- `main.py` 串起了：
  - 四旋翼模型
  - LQR 初始控制器
  - DeePC 控制器
  - 参考轨迹与仿真循环
- `Controllers/deepc.py` 里已经有：
  - `y_ini`
  - `sigma_y`
  - `Y_p g = y_ini + sigma_y` 这种软一致性结构
  - 统一的 `lambda_y` / `lambda_g` 风格正则项
- `quadcopter.py` 中输出定义较集中，适合切换：
  - full output
  - `xyzψ`
  - `xyz`

这意味着：

- 做 measurement-aware regularization 的改动点很明确
- 不需要推翻现有 DeePC 框架
- 但需要自行补：
  - 测量噪声模型
  - yaw bias / yaw drift 注入
  - covariance / residual logging
  - 批量实验入口

因此，从“适不适合作为首篇底座”这个角度看：

- **适合**
- 但必须把它当作研究原型来工程化，而不是直接把当前仓库当论文系统

---

## 2. 三个可做方向

---

# 方向一：固定计算预算的数据选择 DeePC
## 英文可表述
**Computationally Efficient DeePC via Fixed-Budget Data Selection**

## 核心想法
不要每次都使用全部历史数据构造 Hankel，而是：

- 维护较大的历史数据池
- 在线从中选一小部分“最相关”的子轨迹
- 只用这些局部数据构造 DeePC 优化问题
- 让每一步优化规模近似固定

## 这个方向为什么好
它直接打中 DeePC 的经典痛点：

- 数据越多，优化越大
- 优化越大，实时性越差
- 四旋翼控制又要求在线求解稳定、快速

所以这个方向天然有明确卖点：

- 降低平均求解时间
- 降低最坏求解时间
- 保持相近的跟踪精度
- 提高数值稳定性

## 在当前仓库上为什么容易实现
因为当前仓库的 DeePC 管道已经比较清晰：

- 历史输入输出数据
- Hankel 构造
- `g` 决策变量
- CVXPY 求解

你只需要在“构造 Hankel 前”插入一个数据选择器，而不是重写整个系统。

## 可以怎么做
最简可行版本：

1. 把历史数据切成定长子轨迹
2. 每一步根据当前 `y_ini`、参考轨迹 `ref`
3. 选出最相关的 K 条子轨迹
4. 只用这 K 条构建局部 Hankel

### 相似度可以先从简单方法开始
- 当前输出窗口距离
- 当前参考方向相似度
- 最近输入/输出窗口欧氏距离
- 余弦相似度

先不要一开始上复杂机器学习。

## 论文卖点
“在保持跟踪性能的同时显著降低 DeePC 在线优化负担。”

## 优点
- 论文逻辑清楚
- 工程可实现性高
- 容易做消融
- 容易做出漂亮图表
- 最像“首篇能发”的题

## 缺点 / 风险
- 容易被审稿人认为只是工程加速
- 需要非常扎实的实验设计来证明不是简单调参

## 我给的优先级
**最高优先级**

---

# 方向二：输出选择 + 加权正则 DeePC
## 英文可表述
**Output-Aware / Measurement-Aware Regularized DeePC**

## 核心想法
当前仓库把姿态和位置一起作为输出，这会带来问题：

- 姿态测量可能更噪
- yaw 漂移会污染位置跟踪任务
- 不同输出通道可靠性不同

因此可以考虑：

- 只保留 `x, y, z`
- 或保留 `x, y, z, ψ`
- 或者保留全部输出，但对不同输出通道设置不同权重
- 对 slack / regularization 做分通道设计

## 为什么这个方向成立
这不是单纯“删变量”，而是：

- 哪些输出值得强跟踪
- 哪些测量应该弱信任
- 如何在 noisy / drifting measurement 下稳定 DeePC

如果写得好，它可以被包装成：
- measurement-aware DeePC
- anisotropic regularization
- output-channel-sensitive DeePC

## 在当前仓库上为什么容易实现
因为当前仓库里这些量都是显式可改的：

- 输出维度选择
- tracking weight
- regularization
- slack penalty

这意味着：
- 代码改动小
- 很容易批量实验
- 很适合作为首篇的“稳妥路线”

## 最佳搭配
这个方向最好不要单独做，而是和下面这个结构一起：

- 内环稳住 yaw / attitude
- 外环 DeePC 做位置
- 外环做 output-aware regularization

这样论文会更像一个完整系统，而不是单纯试超参数。

## 优点
- 实现难度低
- 容易稳定出结果
- 逻辑比较自然
- 可直接改善当前仓库的核心缺陷

## 缺点 / 风险
- 创新性相对弱
- 如果只做 full output vs xyz 的比较，容易显得像经验调参
- 需要包装成“measurement-aware design”才更像论文

## 我给的优先级
**第二优先级**

---

# 方向三：模式感知 / 数据银行 DeePC
## 英文可表述
**Mode-Aware DeePC / Multi-Bank Data-Driven Predictive Control**

## 核心想法
不是用单一历史数据集构造 Hankel，而是把数据按工况分组：

- hover
- x 向平移
- y 向平移
- figure-8 某些阶段
- yaw 偏差大 / 小
- 大速度 / 小速度

在线时：

- 先根据当前状态判断属于哪种模式
- 再从对应数据银行中选数据
- 再做 DeePC

## 为什么它有意义
因为当前仓库最大问题之一就是：

- 随着 yaw 漂移或工况变化，旧数据和当前动力学越来越不匹配

模式分库本质上是在承认：
“单一全局数据集对整个非线性飞行包线的代表性有限。”

## 优点
- 概念上更像论文
- 比单纯参数调节更有“方法感”
- 能更自然地解释为何在不同飞行阶段表现不同

## 缺点 / 风险
- 实现复杂度更高
- 实验矩阵会显著膨胀
- 很容易陷入“模式怎么分才合理”的调试黑洞
- 对首篇论文来说，风险偏高

## 我给的优先级
**第三优先级**

---

## 3. 我的综合建议

### 重要更新

下面这一节保留的是**早期建议**，它是在“可能仍然考虑计算预算和更强工程部署动机”的前提下给出的。

在后续约束明确为：

- 不做实机
- 纯仿真优先
- 不把在线求解时间当主卖点

之后，最终推荐已更新为：

- **方向二升级为主线**
- 具体做法不是简单 output selection，而是：
  - **measurement-aware / covariance-aware regularized DeePC**
- 方向一不再作为首篇 headline，只保留为后续增强或附加消融
- 方向三继续不建议作为首篇主线

### 最推荐的主线
**先做方向二作为平台修正，再做方向一作为论文主贡献。**

也就是：

### 第一步：先修平台
- 引入简单 inner loop 或简单 yaw stabilizer
- 把 DeePC 的主任务缩成外环位置控制
- 输出优先考虑 `x, y, z` 或 `x, y, z, ψ`

### 第二步：再做论文贡献
- 在这个更稳定的外环 DeePC 上
- 加入固定预算数据选择机制
- 做性能—计算量对比

### 这样做的原因
如果你直接在当前仓库上做数据选择，但平台本身因为 yaw 漂移就容易乱，那么你很难分清：

- 性能变化到底是算法引起的
- 还是平台结构本身导致的

所以最稳的方法是：

**先把底座修平，再叠一个小贡献。**

---

## 4. 不建议的路线

### 不建议 1：继续主力投入 Gazebo / PX4
原因：

- 时间成本高
- 很多时间花在联调和接口
- 论文贡献反而不清晰
- 2025 已经有人把在线更新 DeePC 放进 ROS + PX4 SITL 做过验证

如果只是为了论文首发，继续重投入 Gazebo/PX4 性价比太低。

### 不建议 2：第一篇就做“大理论创新”
比如：
- 全新 DeePC 理论框架
- 完整稳定性证明
- 复杂多层 adaptive 结构

原因：
- 周期长
- 收敛慢
- 风险高
- 不适合“先发一篇”的目标

### 不建议 3：同时做三个贡献
比如同时做：
- 输出选择
- 数据选择
- 在线更新
- 模式分库
- 鲁棒正则

这样很容易每个点都浅，最后都讲不清。

---

## 5. 建议的论文题目候选

### 候选 1（最推荐）
**Computationally Efficient DeePC for Quadcopter Outer-Loop Tracking via Fixed-Budget Data Selection**

### 候选 2
**A Lightweight DeePC Framework for Quadcopter Position Tracking with Output-Aware Regularization**

### 候选 3
**Benchmarking and Improving Regularized DeePC for Quadcopter Tracking under Limited Computation Budgets**

### 候选 4（更偏模式分库）
**Mode-Aware DeePC for Quadcopter Trajectory Tracking under Varying Flight Conditions**

---

## 6. 最小实验矩阵

## 基线方法
至少包含：

- LQR
- 线性 MPC
- 原始 regularized DeePC（repo baseline）
- 你的改进版 DeePC

## 轨迹场景
至少包含：

- hover
- step
- box
- figure-8

## 扰动 / 失配
至少包含：

- 测量噪声增大
- 质量变化
- 推力系数扰动
- 恒定外扰 / 简单风扰

## 指标
至少报告：

- 位置 RMSE
- 最大瞬时误差
- 控制输入幅值 / 平滑性
- 成功率
- 平均求解时间
- 95% 分位求解时间
- 最大求解时间

## 消融实验
### 如果选方向一
- 数据预算 K：20 / 40 / 80 / full
- 选择准则：random / nearest-output / nearest-reference / 你的方法
- 预测时域 N：10 / 15 / 25
- `T_ini`：4 / 6 / 8

### 如果选方向二
- 输出集合：`xyz` / `xyzψ` / 全输出
- 正则方式：统一权重 / 分通道加权
- 是否有 yaw stabilizer：有 / 无

### 如果选方向三
- 模式数
- 模式划分规则
- 模式误判影响
- 模式切换平滑性

---

## 7. 给多个 subagent 的建议分工

### Agent A：文献核查 agent
任务：
- 查四旋翼 / UAV 上已有 DeePC 论文
- 查 2021 real quadcopter DeePC
- 查 2025 online data-updated quadrotor DeePC
- 查 contextual sampling / computationally efficient DeePC

输出：
- 文献表
- 每个方向的新意风险
- 哪个方向最不撞车

### Agent B：仓库结构 agent
任务：
- 深读 `DeePC_Quadcopter` 仓库
- 识别：
  - 数据流
  - Hankel 构造位置
  - output 定义位置
  - solver time 记录点
  - 批量实验需要改哪些文件

输出：
- 代码结构摘要
- 改造优先级清单
- 最小改动路线

### Agent C：算法方案 agent
任务：
- 分别细化三个方向的技术方案
- 给出每个方向的最小可实现版本
- 预判难点、失败模式、实现风险

输出：
- 三个方向各 1 页技术草案
- 推荐首发路线

### Agent D：实验设计 agent
任务：
- 设计 benchmark
- 列出基线、轨迹、扰动、指标、消融
- 判断哪些实验最能打动审稿人

输出：
- 实验矩阵
- 图表清单
- 每个实验对应要回答的问题

### Agent E：审稿人视角 agent
任务：
- 从 novelty、method、experiment 三个角度挑刺
- 判断每个方向最容易被打回的理由
- 给出防守策略

输出：
- 审稿意见模拟
- 可能 rebuttal 点
- 需要补强的实验

---

## 8. 最终推荐（供你拍板）

### 当前状态说明

这部分同样属于早期“待拍板版本”。

**截至 2026-04-01，拍板已经更新：**

- 当前最终推荐不再是“方向二打底 + 方向一主贡献”
- 当前最终推荐是：
  - **以 direction 2 为核心，做一个更窄、更可防守的 measurement-aware regularized DeePC 论文**
- 最终可执行版本已整理在：
  - [recommended_measurement_aware_deepc_direction.md](/home/roxy/deepc-paper/paper/docs/overview/recommended_measurement_aware_deepc_direction.md)

如果目标是：

### “先发一篇”
建议路线：

**方向二打底 + 方向一做主贡献**

即：

- 先把控制任务收缩为外环位置控制
- 引入简单 yaw stabilizer 或 inner loop
- 再做固定预算数据选择 DeePC
- 最后用充分 benchmark 支撑论文

### 不推荐作为首篇主线
- 直接继续重做 Gazebo / PX4
- 直接做模式分库大系统
- 同时做多个贡献点

---

## 9. 一句话结论

**最可行的首篇路线不是“做更大的四旋翼 DeePC 系统”，而是“在轻量稳定底座上，做一个计算效率或测量鲁棒性上的小而清晰改进，并用系统 benchmark 把它讲透”。**
