# DeePC Quadcopter 计划总入口

如果你是新 agent，建议先看：

- [最终方向总览](/home/roxy/deepc-paper/docs/overview/recommended_measurement_aware_deepc_direction.md)
- [工程地图](/home/roxy/deepc-paper/docs/engineering/ENGINEERING_MAP.md)

这组文件把后续工作拆成四部分：

- [01_engineering_experiment_plan.md](/home/roxy/deepc-paper/docs/plans/01_engineering_experiment_plan.md)：工程与实验推进
- [02_paper_writing_plan.md](/home/roxy/deepc-paper/docs/plans/02_paper_writing_plan.md)：论文定位、结构与写作
- [03_milestones_and_exit_criteria.md](/home/roxy/deepc-paper/docs/plans/03_milestones_and_exit_criteria.md)：阶段里程碑、停机条件、转向条件
- [04_experiment_matrix.md](/home/roxy/deepc-paper/docs/plans/04_experiment_matrix.md)：主实验、消融、控制实验矩阵

## 当前拍板版本

当前主线题目先按下面这个工作名推进：

**Measurement-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking**

核心假设：

- 在四旋翼外环 DeePC 中，不同测量通道的可信度不一致。
- `uniform regularization` 会被低可信度通道拖坏。
- 基于测量协方差或残差统计的分组加权 regularization，能在 `yaw drift / anisotropic noise / parameter mismatch` 下更稳。

## 执行顺序

建议按下面顺序推进，不要一开始同时铺太大：

1. 先看 [01_engineering_experiment_plan.md](/home/roxy/deepc-paper/docs/plans/01_engineering_experiment_plan.md)，把平台和 baseline 跑通。
2. 再看 [04_experiment_matrix.md](/home/roxy/deepc-paper/docs/plans/04_experiment_matrix.md)，只做最小主实验。
3. 主实验成立后，再进入 [02_paper_writing_plan.md](/home/roxy/deepc-paper/docs/plans/02_paper_writing_plan.md) 的正式写作节奏。
4. 随时用 [03_milestones_and_exit_criteria.md](/home/roxy/deepc-paper/docs/plans/03_milestones_and_exit_criteria.md) 判断要不要收缩问题。

## 当前最优先事项

- 把 `DeePC_Quadcopter` 整理成可重复实验入口。
- 先复现 `LQR / linear MPC / original regularized DeePC`。
- 再补测量模型层：`yaw bias / yaw drift / anisotropic noise`。
- 最后才上第一版 `measurement-aware regularization`。
