# 异步 / 延迟 / 丢包感知 DeePC：预开题包

## 1. 方向一句话定义

将当前四旋翼外环 DeePC 的研究主线，从“measurement-aware weighting”转为：

**面向异步、延迟与丢包观测链的 DeePC 设计，用于轻量仿真中的四旋翼外环轨迹跟踪。**

更具体一点可以表述为：

- **Asynchronous-Measurement-Aware DeePC for Quadcopter Outer-Loop Tracking**
- **Delay / Dropout-Aware DeePC with Time-Aligned History Windows**
- **DeePC for Quadcopter Tracking under Delayed and Missing Measurements**

---

## 2. 为什么现在优先做这条线

这条线之所以适合作为下一轮唯一主线，有 4 个原因：

1. **和当前 repo 连续**
   - 你当前仓库已经暴露出 measurement path / observation path 的问题；
   - 继续沿“观测链问题”推进，比重新换一个完全陌生方向更省成本。

2. **不是继续修 weighting**
   - 旧主线的问题是：容易沦为“再调一个权重”；
   - 新主线把问题改成“观测可用性与时间对齐”，问题结构发生了变化。

3. **适合轻量仿真**
   - 不需要 Gazebo / PX4 / 实机；
   - 自写动力学和轻量轨迹跟踪实验就足够支撑最小验证。

4. **即使失败也容易体面降级**
   - 如果方法没有赢，可以退成：
     - analysis paper
     - benchmark paper
     - negative result with clear setup guidance

---

## 3. 建议的主问题

### 推荐主问题版本
> 在四旋翼外环 DeePC 中，当输出测量存在固定延迟、随机丢包或异步采样时，能否通过时间对齐和缺测感知的数据栈构造，提升闭环跟踪稳定性与鲁棒性？

### 研究问题拆分
1. 当前标准 DeePC 在 delayed / missing measurements 下会如何退化？
2. 这种退化主要来自：
   - 历史窗口错位？
   - y_ini 不一致？
   - consistency fitting 被脏观测污染？
3. 是否存在一种**最小方法**，能比“直接忽略问题”更稳？

---

## 4. 最小方法定义（首发只做一个）

## 推荐唯一首发方法
**Time-Aligned History Window DeePC**

### 核心思想
当前 DeePC 依赖：
- 最近一段输入 / 输出历史
- 当前观测窗口 `y_ini`
- Hankel / 数据栈一致性

如果观测存在延迟、异步或丢包，那么直接把“最新可见观测”塞进标准 DeePC，会导致：
- 历史窗口与当前真实时刻错位
- `y_ini` 包含时间不一致样本
- consistency fitting 被破坏

因此最小方法是：

### 方法 M1：时间对齐历史窗口
- 对观测时间戳显式建模
- 只使用与当前控制时刻对齐的历史窗口
- 对延迟样本做 index realignment
- 丢失观测位置用 mask 标记，而不是伪造正常样本

这比纯 weighting 更像真正方法，因为：
- 改的是 DeePC 数据栈构造方式
- 不是简单调 tracking weight
- 不是再调 regularization 系数

---

## 5. 只允许考虑的候选最小变体

首轮开发建议只保留下面 3 个变体，不要再扩：

### Variant A：Fixed-delay aligned DeePC
适用：
- 固定 1-step / 2-step 输出延迟

做法：
- 构造延迟补偿后的 `y_ini`
- 历史窗口按 delay 对齐

### Variant B：Dropout-masked DeePC
适用：
- 部分时刻观测缺失

做法：
- 对缺测观测加 mask
- 只在可用观测维度上做 consistency fitting 或替代处理

### Variant C：Asynchronous-window DeePC
适用：
- 多个输出通道并非同步刷新
- 或输出时间戳不完全一致

做法：
- 显式保存 measurement timestamp
- 基于最近一致可用窗口构建 `y_ini`

### 首轮推荐顺序
1. Variant A
2. Variant B
3. Variant C

原因：
- A 最简单，最容易快速判断方向是否值得继续
- B 第二简单
- C 最像论文，但工程复杂度也最高

---

## 6. 最小代码改动点

按你当前 repo 习惯，优先检查这几类位置：

### 1. measurement 生成与传递链
重点：
- `Simulation.simulate()` 中 measurement 的采样、记录、传递逻辑
- controller 真正消费的是哪一版 measurement

### 2. DeePC 控制器的数据栈构造
重点：
- `compute_input()` 内 `y_ini` 构造逻辑
- 历史输入/输出窗口截取逻辑
- Hankel / consistency fitting 所用观测序列

### 3. 实验配置与 scenario 注入
重点：
- 如何注入：
  - fixed delay
  - Bernoulli dropout
  - asynchronous update
- 是否能通过 config/CLI 切换

### 4. 结果汇总
重点：
- 是否能汇总：
  - RMSE
  - final error
  - pass rate
  - delay/dropout level

---

## 7. 最小实验矩阵

首轮不要做大矩阵。

### 轨迹
- step
- figure8

### 场景
- nominal
- fixed 1-step delay
- fixed 2-step delay
- dropout 10%
- dropout 20%

如果算力不足，最小可进一步压缩为：
- nominal
- 1-step delay
- dropout 20%

### 控制器
- uniform regularized DeePC
- 你的新方法（先只测一个变体）
- 如有余力，再加：
  - xyz-only DeePC（辅助参考）

### 种子
- 先 3 seeds smoke test
- 再 5 seeds
- 如果出现稳定趋势，再上 10 seeds

### 指标
- position RMSE
- final_position_error_norm
- success / pass rate
- 如有意义可加：
  - average control effort
  - maximum position error

---

## 8. 第一轮不该做什么

为了防止再次发散，首轮开发严禁：

- 不要继续做 measurement-aware weighting tweak
- 不要同时做多个方法组合
- 不要同时把 async + fault + regime 全做
- 不要扩到 wind / mass variation / obstacle avoidance
- 不要碰 PX4 / Gazebo / 实机
- 不要先追求漂亮结果再解释机制

---

## 9. 最可能失败的地方

### 失败模式 1：只是换了一种观测预处理
风险：
- 审稿人会说你只是做了 preprocessing，而不是 DeePC 方法

应对：
- 必须强调改的是 `y_ini` / history stack / consistency construction
- 而不是简单滤波

### 失败模式 2：delay 下有效，dropout 下无效
风险：
- 方法适用范围过窄

应对：
- 首篇可以先只主打 fixed-delay
- dropout 作为次要扩展

### 失败模式 3：nominal 下反而退化
风险：
- 方法不够“free lunch”

应对：
- 把 nominal degradation 当作重要约束
- 不允许为了坏场景收益而明显毁掉 nominal

### 失败模式 4：效果不如简单 heuristic
风险：
- 复杂方法不如直接丢弃坏样本/简单延迟补偿

应对：
- 需要显式和简单 heuristic 比较
- 不能只和最弱 baseline 比

---

## 10. 如果首轮结果不理想，如何体面降级

如果方法没有形成稳定优势，可以降级为：

### 降级路线 A：analysis paper
主题：
- DeePC 对 delay / dropout / async measurement 的敏感性分析

### 降级路线 B：benchmark / implementation note
主题：
- 轻量四旋翼 DeePC 仿真中，观测时间对齐与缺测处理的实现影响

### 降级路线 C：negative result
主题：
- 某类简单 alignment/masking 机制不足以稳定改善 DeePC

所以这条线即使方法失败，也比继续做 weighting tweak 更不容易“全盘归零”。

---

## 11. 下一轮建议只回答的 3 个问题

在真正开发前，只让 agent 先回答这 3 个问题：

1. 当前 repo 里，观测的时间索引和 controller 消费索引是否已经明确？
2. 固定 1-step delay 能否作为最小开发目标？
3. 首轮最小方法到底是：
   - aligned y_ini
   - masked history
   - 还是 delayed consistency fitting？

只有这 3 个问题回答清楚，再开始正式改代码。

---

## 12. 推荐的一句话题目候选

### 保守版
**Delay-Aware DeePC for Quadcopter Outer-Loop Tracking in Lightweight Simulation**

### 更方法版
**Time-Aligned History Window DeePC under Delayed and Missing Measurements**

### 更完整版
**Asynchronous-Measurement-Aware DeePC for Quadcopter Outer-Loop Tracking**

---

## 13. 最终建议

当前不要再继续同时展开多个新方向。

**下一轮唯一主线：**
**异步 / 延迟 / 丢包感知 DeePC**

**首轮唯一方法：**
**Time-Aligned History Window DeePC**

**首轮唯一目标：**
先证明“标准 DeePC 在 delayed / missing measurements 下会明显退化，而时间对齐后的历史窗口构造能带来更稳定的闭环表现”。

只要这个最小命题能站住，后续再考虑：
- dropout mask
- asynchronous multi-rate
- fault/degradation 作为备胎方向
