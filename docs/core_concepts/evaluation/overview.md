# 评估概览

评估是任何后训练工作流中的关键组成部分。它是衡量进展的标准。在开始后训练之前就建立稳健的评估方法与基准，对于确保结果有意义且可复现至关重要。

本节为不同类型的视频生成模型提供评估方法。评估策略会根据所训练模型的类型而不同，本指南可帮助你根据模型类型与使用场景选择合适的方法。

## 指标家族

- **定性视频质量（Predict）**
  - **FID** — 在 Inception 特征空间中通过 Fréchet distance 衡量图像真实感/多样性（越低越好）
  - **FVD** — 在视频特征上通过 Fréchet distance 衡量时空质量（外观 + 运动）
- **几何一致性（Predict）**
  - **Sampson Error** — 点到极线距离的一阶近似
  - **TSE/CSE** — 面向多视角视频的时间一致性与跨视角一致性
- **基于 VLM 的评估**
  - **Cosmos Reason** — 评估物理合理性、因果/时间推理；可用作 critic 或 reward model。
- **Transfer/Control 质量**
  - **Blur SSIM、Canny-F1、Depth RMSE、Seg mIOU、Dover** — 衡量控制信号保真度与技术质量。

## 推荐工作流

1. 指定评估划分以及帧/片段采样策略。
2. 根据模型类型（Predict vs Transfer/Control）与目标选择指标。
3. 对 Pred ↔ GT 对齐预处理（分辨率、裁剪、fps）。
4. 先运行核心指标（FID/FVD 或控制指标），再根据需要进行几何检查（TSE/CSE）。
5. 为物理与推理质量补充 VLM 分析（critic/reward）。
6. 报告均值、方差/置信区间以及精确配置，以确保可复现性。

## 模型→指标映射

- Predict（生成式）→ [evaluation_predict.md](evaluation_predict.md)
- Transfer/Control（ControlNet）→ [evaluation_transfer.md](evaluation_transfer.md)
- Reason Reward/Critic（VLM）→ [reason_as_reward.md](reason_as_reward.md)

