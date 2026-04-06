# DeePC + 无人机 + 轻量仿真：本轮方向筛选总结

## 1. 本轮结论一句话版

本轮方向筛选后，**最推荐继续推进的唯一主线**是：

**异步 / 延迟 / 丢包感知 DeePC（delay / dropout / asynchronous-measurement-aware DeePC）**

**备胎主线**是：

**fault / degradation-aware DeePC**

**暂不推荐优先推进**的方向包括：

- regime / maneuver-conditioned DeePC
- safety-filtered / obstacle-aware DeePC
- multi-UAV / formation DeePC
- 再做任何纯 regularization tweak
- 再做任何 measurement-aware weighting tweak
- 一开始就做组合方法

---

## 2. 为什么要换主线

之前的 measurement-aware / covariance-aware regularized DeePC 主线，已经有比较明确的阶段性结论：

1. **问题被成功造出来了**
   - `uniform DeePC` 在 `yaw_drift`、`anisotropic_noise` 等坏场景下确实会退化。

2. **当前改进方案没有形成稳定、可复现、可防守的整体改进**
   - `measurement_noise`
   - `robust_residual_stats`
   - 以及其他 residual-statistics 风格候选  
   都没有在冻结核心矩阵上稳定优于 `uniform`。

3. **实验公平性还存在硬问题**
   - `LQR / MPC` 与 `DeePC` 的 observation path 不一致
   - `Simulation` 与 `DeePC.compute_input()` 间还存在额外 measurement sampling 问题

因此，继续沿原主线细修小补，风险很高，且容易再次落入“局部改善但主方法 claim 立不住”的困境。

所以这轮的目标改成：
**寻找新的、可拓展、低撞题风险、适合轻量仿真的方法论文主线。**

---

## 3. 本轮筛选后的方向排序

### Top 1：异步 / 延迟 / 丢包感知 DeePC
英文可表述：
- **Asynchronous-Measurement-Aware DeePC**
- **Delay / Dropout-Aware DeePC for Quadcopter Outer-Loop Tracking**

#### 为什么排第一
这条线最像“真正换了问题结构”，而不是继续在旧 regularization 上缝补。

它的优势：

- 和当前 repo 的测量链问题高度连续
- 不需要切换到 Gazebo / PX4 / 实机
- 方法核心是“观测可用性、时间对齐、缺测处理”，不是“再调一个权重”
- 即使方法最终没成功，也很容易退成一篇 analysis / benchmark / negative result 型论文
- 在目前调研结果下，它比一般性的 measurement corruption 主题更具体，撞题风险更低

#### 这条线适合的最小方法方向
可以考虑：
- masked history DeePC
- time-aligned history window DeePC
- delayed-output compensation inside DeePC data stack
- dropout-aware consistency fitting
- asynchronous multi-rate measurement handling

#### 当前建议
把这条线作为**唯一主线**，不要一开始就和其他方向并行开发。

---

### Top 2：fault / degradation-aware DeePC
英文可表述：
- **Fault-Aware DeePC**
- **Degradation-Aware DeePC for Quadcopter Tracking**

#### 为什么排第二
这条线仍然有方法潜力，而且也适合轻量仿真。

它的优点：
- 和无人机控制任务天然契合
- 容易定义退化场景
- 论文故事比较完整
- 与现有轻量仿真底座兼容

但我对它的判断比一些 agent 输出要更保守半档。

#### 风险
- fault-tolerant control 本身文献很多
- degradation-aware / fault-aware 的 general control 思路并不新
- 如果最后只是“nominal bank + degraded bank”或简单 bank switching，很容易像把已有思路换到 DeePC 上

#### 当前建议
把它作为**备胎主线**，而不是和 Top 1 并行推进。

---

### Top 3：regime / maneuver-conditioned DeePC
英文可表述：
- **Regime-Aware DeePC**
- **Maneuver-Conditioned DeePC**
- **Mode-Aware DeePC for Quadcopter Tracking**

#### 为什么只排第三
这条线乍看合理，但相邻思路太近了：

- contextual sampling
- bank selection
- local Hankel
- gain scheduling
- operating-region partition

这些 general DeePC / DPC 思路已经很多，如果再搬到 quadrotor 轻量仿真里，容易形成“有实现、有结果，但方法味道不够新”的局面。

#### 当前建议
保留为远期备选，但**不建议作为下一轮默认主线**。

---

## 4. 其他方向为什么不推荐优先推进

### safety-filtered / obstacle-aware DeePC
优点：
- 方法感强
- 容易讲“安全性”故事

缺点：
- 工程负担明显更高
- 实验设计复杂
- 对当前 repo 连续性较差
- 首篇容易做重

所以不适合当前阶段优先做。

---

### multi-UAV / formation DeePC
优点：
- 论文故事大
- 看起来很“高级”

缺点：
- 多机耦合、通信、协同、分布式控制复杂度都很高
- 工程风险高
- 很容易超出“首篇方法论文”的可控范围

因此当前不推荐。

---

### 再做纯 regularization tweak / measurement-aware tweak
不推荐原因最明确：

- 你已经在这条线上做过一轮较完整尝试
- 结果没有形成稳定主张
- 继续做极容易再次滑回“局部改善但无整体结论”
- 容易沦为“又一个 tweak”

这条线现在应该明确降权。

---

### 一开始就做组合方法
例如：
- fault-aware + async-aware
- regime-aware + data bank
- async-aware + measurement weighting
- safety filter + DeePC variant

当前一律不推荐。

原因：
- 组合方法会大幅增加解释复杂度
- 很难判断到底哪个组件有效
- 首篇阶段最应该追求的是“单一主线 + 单一最小方法”

---

## 5. 本轮最关键的内部修正

本轮筛选中，需要注意两个措辞修正：

### 修正 1：fault / degradation-aware 不是低风险空白区
更稳妥的理解应是：

- **新意风险：中等**
- **实现风险：中等**
- **可做，但不能乐观**

它可以作为备胎，但不应被理解为“明显没人做过”。

---

### 修正 2：measurement corruption 大类已拥挤，但 async/delay/dropout 子问题仍可做
更稳妥的表述应是：

- **广义 measurement corruption / noisy measurement DeePC：已较拥挤**
- **quadrotor + DeePC + asynchronous / delayed / missing measurements：仍可作为更具体的新问题**

所以后续命名绝不能再泛泛写成：

- measurement-aware DeePC

而应该写成更窄的主线，例如：

- **asynchronous-measurement-aware DeePC for quadrotor outer-loop tracking**
- **delay/dropout-aware DeePC with masked and time-aligned history windows**

---

## 6. 当前最推荐的项目定位

### 唯一主线
**异步 / 延迟 / 丢包感知 DeePC**

### 备胎主线
**fault / degradation-aware DeePC**

### 暂不推进
- regime / maneuver-conditioned
- safety-filtered / obstacle-aware
- multi-UAV
- 任何纯 weighting tweak
- 任何组合方法

---

## 7. 下一步最建议做什么

不是继续同时搜 5 个方向，  
而是立刻围绕 **Top 1 异步观测主线** 做“预开题级”输出：

1. 一句话题目
2. 最小方法定义
3. 最小代码改动点
4. 最小实验矩阵
5. 最可能失败模式

这样可以尽快判断它是不是值得正式开发，而不是继续停留在方向筛选层。

---

## 8. 给下一轮开发的硬规则

后续无论用 Codex 还是 subagent，都建议加上这条规则：

> **如果一个方向的新意风险和实现风险同时不低于中高，则默认不能排第一。**

这条规则能自动压住一些“听起来很强但容易做死”的方向。

---

## 9. 最终结论

本轮方向筛选后，项目不应继续死磕旧的 measurement-aware weighting 主线。

更稳妥的路线是：

- **主线：异步 / 延迟 / 丢包感知 DeePC**
- **备胎：fault / degradation-aware DeePC**
- **原则：先做单一主线 + 单一最小方法，不做组合**

这比继续做 regularization 小修小补，更有希望形成一篇真正的方法论文，同时仍保持轻量仿真、非 Gazebo 的开发风格。
