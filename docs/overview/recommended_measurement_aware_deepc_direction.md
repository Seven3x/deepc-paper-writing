# 推荐方向：用于四旋翼外环位置跟踪的 Measurement-Aware Regularized DeePC

## 0. Agent Handoff

这份文档是**当前最终执行版本**，用于让新 agent 打开后立刻理解：

- 现在真正拍板的研究方向是什么
- 为什么不是其他两个方向
- 当前工程底座是什么
- 接下来最应该做哪些改造

如果你是新 agent，请按下面顺序理解任务：

1. 这篇论文的主线已经拍板为：
   - **Measurement-Aware / Covariance-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking**
2. 不要再把主线放在：
   - fixed-budget data selection
   - mode-aware / multi-bank DeePC
3. 如果需要理解早期分歧和三方向比较，请回看：
   - [deepc_quadcopter_three_directions_summary.md](/home/roxy/deepc-paper/docs/overview/deepc_quadcopter_three_directions_summary.md)
4. 如果需要开始工程实现，请优先围绕：
   - 测量模型
   - 输出配置
   - covariance-aware regularization
   - 批量实验
5. 如果需要直接进入代码，请继续看：
   - [ENGINEERING_MAP.md](/home/roxy/deepc-paper/docs/engineering/ENGINEERING_MAP.md)

### 当前已知工程底座

工程底座是：

- `MartinPetre/DeePC_Quadcopter`

对后续实现最关键的已知事实：

- `main.py` 串起了 quadcopter、LQR、DeePC、参考轨迹和仿真循环
- `Controllers/deepc.py` 已有：
  - `y_ini`
  - `sigma_y`
  - 软一致性约束
  - 统一 regularization 结构
- `quadcopter.py` 已有集中式输出定义，适合改：
  - full output
  - `xyzψ`
  - `xyz`

### 新 agent 最应该马上做的事

如果你的任务是继续推进工程或论文，请优先处理：

1. 给测量层补：
   - anisotropic measurement noise
   - yaw bias
   - yaw drift
   - covariance / residual logging
2. 给 DeePC 补：
   - uniform / manual group-wise / covariance-aware regularization
3. 给实验框架补：
   - 多轨迹
   - 多扰动
   - 多随机种子
   - 自动汇总结果

### 不该再浪费时间的方向

- 不要把 computational efficiency 当主卖点
- 不要把 online data update 当首篇 headline
- 不要把 mode-aware bank switching 当首篇主线
- 不要把简单 `xyz-only` output pruning 当主要创新

---

## 1. 结论

在当前约束下：

- 不做实机
- 以轻量仿真为主
- 不把在线计算时间当核心卖点

最合适的首篇论文方向不是 fixed-budget data selection DeePC，也不是 mode-aware / multi-bank DeePC，而是：

**Measurement-Aware / Covariance-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking**

更具体地说，建议将论文主线收敛为：

- 外环位置跟踪任务
- 输出通道可信度不均衡
- yaw drift / yaw bias / 各向异性测量噪声导致 DeePC data-consistency fitting 被污染
- 基于测量协方差或残差统计的分组加权 regularization / slack design

---

## 2. 为什么这是最合适的方向

### 2.1 不再推荐方向一作为主线

原先方向一“固定预算数据选择 DeePC”最大的卖点是：

- 降低平均求解时间
- 降低最坏求解时间
- 提高实时部署可行性

但在当前设定里：

- 不做实机
- 固定控制频率
- 不把求解时间当主结果

那么这条线的核心说服力会明显下降。

同时，2025 年已经出现非常接近的在线数据选择 DeePC 工作：

- Näf et al., *Choose Wisely: Data-Enabled Predictive Control for Nonlinear Systems Using Online Data Selection*, 2025  
  https://arxiv.org/abs/2503.18845
- Beerwerth and Alrifaee, *Less is More: Contextual Sampling for Nonlinear Data-Enabled Predictive Control*, 2025  
  https://arxiv.org/abs/2503.23890

因此，如果只在纯仿真中做“数据选择 DeePC”，很容易变成：

“没有实机、没有实时性结果、也没有明显超出已有 nonlinear DeePC online selection 工作的创新。”

### 2.2 方向二最贴合当前问题

当前四旋翼 DeePC 平台的核心问题，本来就集中在：

- 外环任务本质上主要关心位置
- 姿态 / yaw 通道测量质量未必和位置通道一样可靠
- yaw 漂移和姿态误差会污染位置跟踪
- 统一 regularization 会把不同质量的输出通道一视同仁

所以方向二并不是“重新找题”，而是**直接修正当前平台最核心的不合理处**。

### 2.3 方向三不适合作为首篇主线

mode-aware / multi-bank DeePC 虽然概念上更像方法论文，但问题在于：

- 最近已经有 gain-scheduled / local-bank / context-conditioned DeePC 近邻工作
- 模式划分不自然
- 实验矩阵大幅膨胀
- 很容易陷入 mode design 和 switching heuristics 黑洞

对于首篇纯仿真工作，这条线风险偏高。

---

## 3. 文献依据

### 3.1 四旋翼 DeePC 的关键锚点

最重要的直接锚点是：

- Elokda et al., *Data-enabled predictive control for quadcopters*, 2021  
  https://pmc.ncbi.nlm.nih.gov/articles/PMC9291934/

这篇工作已经做了：

- quadcopter outer-loop DeePC
- cascaded architecture
- yaw mismatch / calibration 问题分析
- 输出通道数量敏感性比较

更重要的是，这篇论文实际上已经点出了一个明确 open thread：

- orientation / yaw 类输出更噪
- 丢弃这类通道有时更稳
- 这个问题可以通过**基于测量噪声协方差的加权 penalization**处理

这意味着：

- “只保留 `xyz` 更好”不是新点
- “yaw 更 noisy 所以权重小一点”也不是新点
- 真正还没被系统做好的，是把这个直觉做成一个**measurement-structured DeePC design**

### 3.2 robust / regularized DeePC 主干已较成熟

相关主干文献包括：

- Coulson et al., *Regularized and Distributionally Robust Data-Enabled Predictive Control*, 2019  
  https://arxiv.org/abs/1903.06804
- Huang et al., *Quadratic Regularization of Data-Enabled Predictive Control*, 2021  
  https://arxiv.org/abs/2012.04434
- Huang et al., *Robust Data-Enabled Predictive Control*, 2021  
  https://arxiv.org/abs/2105.07199
- Breschi et al., *On the impact of regularization in data-driven predictive control*, 2023  
  https://arxiv.org/abs/2304.00263
- Chiuso et al., *Harnessing uncertainty for a separation principle in direct data-driven predictive control*, 2023/2025  
  https://arxiv.org/abs/2312.14788

这些文献说明：

- “regularization 有用”不是新意
- “uncertainty-aware DDPC”也不是空白
- 但**将输出通道质量结构显式写进 DeePC 的 regularization / consistency fitting**，仍然没有被标准化为成熟路线

这就是当前最可做、但又不能做太大的空档。

---

## 4. 推荐的问题定义

建议的问题定义为：

**在四旋翼外环位置跟踪任务中，研究 heterogeneous measurement quality 对 DeePC 闭环性能的影响，并设计 measurement-aware regularization 以提升 yaw drift、各向异性噪声和参数失配下的鲁棒性。**

更具体地说：

- 研究对象：quadcopter outer-loop position tracking
- 主要输出：`xyz` 或 `xyzψ`
- 不再尝试端到端接管完整姿态控制
- 不主打计算效率
- 不主打在线数据更新
- 不主打 mode switching

---

## 5. 最推荐的方法版本

### 5.1 方法总思路

从标准 regularized DeePC 出发，保留：

- tracking cost
- control effort cost
- `g` regularization
- output consistency slack

核心改动只做一处：

把 DeePC 中针对输出一致性误差的统一惩罚，改成**measurement-aware / covariance-aware 加权惩罚**。

### 5.2 推荐主版本

推荐主版本为：

- 输出集合：`xyzψ`
- `xyz` 作为强任务通道
- `ψ` 作为弱可信辅助通道
- 对 `Y_p g - y_ini` 的 softened consistency / slack 使用分组加权

这是当前最推荐的版本，因为它比单纯 `xyz-only` 更像方法论文：

- 不是直接删变量
- 而是在保留辅助信息的同时控制其信任强度
- 更容易防守“这不是简单 output pruning”

形式上可写为：

\[
\min \; J_{\text{track}} + J_{\text{input}} + \lambda_g \|g\|_2^2 + \sigma_y^\top W_\sigma \sigma_y
\]

subject to

\[
Y_p g = y_{\mathrm{ini}} + \sigma_y
\]

其中：

- `W_sigma` 为 block-diagonal weighting matrix
- 最简单版本：
  - `xyz` 一组
  - `ψ` 一组
- 更像论文的版本：
  - `W_sigma \propto \Sigma_y^{-1}`
  - 其中 `\Sigma_y` 来自测量噪声协方差或残差协方差估计

### 5.3 为什么不要把创新点只放在 Q 上

如果只调整 tracking cost 里的 `Q`，审稿人很容易说：

- 这只是普通 MPC 的调权
- 没有 DeePC-specific 的方法意义

因此更合适的写法是：

- 不只是改变 tracking priorities
- 而是改变 DeePC 如何“相信”不同测量通道
- 也就是改变它的 data-consistency fitting 结构

---

## 6. 建议的方法对比版本

建议至少比较以下方法：

- `M0`: LQR
- `M1`: 线性 MPC
- `M2`: 原始 regularized DeePC
- `M3`: `xyz-only` DeePC
- `M4`: `xyzψ + uniform regularization`
- `M5`: `xyzψ + manual group-wise regularization`
- `M6`: `xyzψ + covariance-aware regularization`

其中：

- `M3` 是一个必须打赢的 baseline
- 如果 `M6` 打不过 `M3`，论文会很难成立

---

## 7. 论文主贡献建议

建议将论文主贡献压缩成 3 点：

1. 识别并刻画 quadcopter outer-loop DeePC 中 heterogeneous measurement quality 导致的 data-consistency 失配问题。
2. 提出一种基于 measurement covariance / residual statistics 的 group-wise regularized DeePC。
3. 在纯仿真 benchmark 中验证其在 yaw drift、各向异性测量噪声和参数失配下的鲁棒性提升。

不建议写成主贡献的内容：

- computational efficiency
- online data update
- mode-aware bank switching
- 通用 nonlinear DeePC 理论突破
- 完整稳定性理论

---

## 8. 实验矩阵

### 8.1 主实验

回答的问题：

**measurement-aware regularization 是否比 uniform regularization 更能抗输出失配？**

轨迹至少包含：

- hover
- step
- box
- figure-8

扰动 / 失配至少包含：

- yaw bias
- yaw drift
- anisotropic measurement noise
- mass variation
- thrust coefficient mismatch
- constant wind / disturbance

### 8.2 消融实验

#### 消融 A：输出设置

- full output
- `xyzψ`
- `xyz`

回答的问题：

- 改进是不是只是来自删变量？

#### 消融 B：regularization 类型

- uniform
- manual group-wise
- covariance-informed

回答的问题：

- 改进是不是只是手动调权？

#### 消融 C：协方差失配

- accurate covariance
- underestimated yaw noise
- overestimated yaw noise

回答的问题：

- 方法对协方差估计偏差是否稳健？

### 8.3 随机性要求

建议：

- 每个主要实验至少 5 到 10 个随机种子
- 报告均值和方差
- 不要只放单条“好看轨迹”

---

## 9. 指标设计

至少报告：

- position RMSE
- maximum position error
- success rate / crash-free rate
- control effort
- control smoothness

建议补充：

- 多 seed boxplot
- failure case trajectory
- error CDF

如果不把 runtime 当卖点：

- 求解时间最多放附录
- 不要作为主结果组织论文

---

## 10. 预期的主要审稿人质疑

### 质疑 1

“这不就是调权重吗？”

应对：

- 强调改的是 consistency/slack regularization，不只是 tracking cost
- 权重来自 measurement covariance / residual statistics，不是纯 hand tuning

### 质疑 2

“为什么不直接 estimator + MPC？”

应对：

- 明确论文目标是 lightweight DeePC pipeline 下的 practical robustness
- 不 claim 替代 model-based estimator pipeline

### 质疑 3

“没有硬件验证，结论不够强。”

应对：

- 用更系统的仿真矩阵弥补
- 多轨迹、多扰动、多随机种子、多失效案例

### 质疑 4

“直接 `xyz-only` 不就够了？”

应对：

- 必须把 `xyz-only DeePC` 设为强 baseline
- 证明 measurement-aware `xyzψ` 至少在关键场景下优于简单 pruning

---

## 11. 工程实现建议

基于 `MartinPetre/DeePC_Quadcopter`，最小改造建议如下：

### 11.1 先补测量层

- anisotropic measurement noise
- yaw bias
- yaw drift
- residual logging
- covariance estimation

### 11.2 再补输出配置

- full output
- `xyzψ`
- `xyz`

### 11.3 再改 DeePC 正则项

- uniform regularization
- manual group-wise regularization
- covariance-aware regularization

### 11.4 最后补批量实验入口

- 多轨迹
- 多随机种子
- 多扰动配置
- 自动收集结果表

### 11.5 当前建议的实现优先级

建议严格按这个顺序推进，不要一开始就大改：

1. 先跑通原始仓库并确认 baseline 可复现
2. 再补测量层和输出配置
3. 再做 `xyz-only` 与 `xyzψ + uniform` 的 baseline 对比
4. 再加入 `xyzψ + covariance-aware regularization`
5. 最后再做批量实验和协方差失配分析

这样做的原因是：

- 先验证平台本身
- 再验证“问题现象”是否存在
- 最后再验证“方法改动”是否有效

---

## 12. 给后续 agent 的执行边界

如果你是后续 agent，默认应遵守以下边界：

- 目标是首篇论文，不是完整系统
- 先保守实现，再逐步加复杂度
- 默认优先 `xyzψ` 主版本
- `xyz-only` 是 baseline，不是最终主方法
- 不默认要求 inner loop 大改
- 不默认要求真机或 PX4 / Gazebo 联调
- 不默认追求复杂理论证明

如果后续需要做方向扩张，建议顺序是：

1. covariance-aware regularization
2. covariance misspecification robustness
3. light residual-adaptive weighting
4. data selection 作为附加增强
5. mode-aware / multi-bank 留到后续工作

---

## 13. 最终推荐

当前最推荐的首篇路线可以概括为：

**用轻量四旋翼仿真平台研究外环位置 DeePC 在输出通道质量不均衡下的鲁棒性问题，并通过 covariance-aware regularization 设计，解决 yaw drift / anisotropic measurement noise 对 data-consistency fitting 的污染。**

更简洁的一句话是：

**最值得做的不是“更快的 DeePC”，而是“一个针对四旋翼外环输出失配问题的 measurement-aware regularized DeePC”。**
