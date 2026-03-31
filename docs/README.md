# Docs Index

这套文档分为三层：

- `overview/`
  方向判断与最终拍板
- `plans/`
  执行计划、实验矩阵、里程碑和写作计划
- `engineering/`
  工程目录说明、代码改造落点和实现优先级

## 最重要的两份文档

- [最终方向](/home/roxy/deepc-paper/docs/overview/recommended_measurement_aware_deepc_direction.md)
- [计划入口](/home/roxy/deepc-paper/docs/plans/plan_index.md)

## 阅读顺序

如果你是新 agent：

1. [recommended_measurement_aware_deepc_direction.md](/home/roxy/deepc-paper/docs/overview/recommended_measurement_aware_deepc_direction.md)
2. [plan_index.md](/home/roxy/deepc-paper/docs/plans/plan_index.md)
3. [ENGINEERING_MAP.md](/home/roxy/deepc-paper/docs/engineering/ENGINEERING_MAP.md)
4. 需要理解早期取舍时，再看 [deepc_quadcopter_three_directions_summary.md](/home/roxy/deepc-paper/docs/overview/deepc_quadcopter_three_directions_summary.md)

## 当前拍板

当前最终主线：

**Measurement-Aware / Covariance-Aware Regularized DeePC for Quadcopter Outer-Loop Position Tracking**

当前工程底座：

- [DeePC_Quadcopter](/home/roxy/deepc-paper/DeePC_Quadcopter)

当前不建议作为首篇主线：

- fixed-budget data selection
- mode-aware / multi-bank DeePC
- online data update
- 简单 output pruning
