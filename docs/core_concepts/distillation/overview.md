# 模型蒸馏

模型蒸馏是一项非常强大的技术，可用于构建高效的 “Student” 模型：在大幅降低计算需求的同时，尽可能保留 “Teacher” 模型的质量与能力。借助这一过程，可以在资源受限的环境中部署高性能模型，同时维持输出质量和多样性。

当前广泛使用的蒸馏方式主要有两类：

- **尺寸蒸馏（Size Distillation）**：将大模型压缩为更小的架构，同时尽量保持性能。

- **步数蒸馏（Step Distillation）**：减少推理所需的步数。

本指南重点讨论步数蒸馏——即训练 Student 模型，使其仅通过一次 diffusion step 就能取得与 Teacher 模型相近的结果——而 Teacher 模型则通常需要多步（例如 Cosmos Transfer 1 中的 36 个 diffusion steps）。

在典型的蒸馏工作流中，可以调整多个变量来优化模型效果：

**数据混合（Data Mixture）**：蒸馏训练数据集的构成对蒸馏效果至关重要。目标领域中的视频数据既需要足够规模，也需要足够高质量，才能获得最佳的蒸馏模型性能。数据集应具备多样性、能够代表预期使用场景，并尽量不含会削弱 Student 模型能力的伪影。

**训练策略（Training Strategy）**：蒸馏训练策略的选择取决于数据可用性、任务复杂度和计算预算。蒸馏通常采用多阶段 curriculum training，不同阶段使用针对特定学习目标优化的不同算法。目前支持的方式包括：

- **Knowledge Distillation（KD）**：一种轻量但有效的蒸馏方法，通过对模型输出施加回归损失，让 Student 模型与 Teacher 模型对齐。该方法需要先使用 Teacher 模型生成完整的输入输出对合成数据集，因此非常适合作为初始训练阶段，用于建立模型之间的基础对齐。
- **Improved Distribution Matching Distillation（DMD2）**：一种分布匹配方法，结合对抗训练与 variational score distillation，以保留输出的多样性和质量。该方法需要一个多样化的真实视频数据集，并且要同时训练多个模型（Student、Teacher、fake score network 和 discriminator），因此会带来更高的显存需求。参见 [DMD2 paper](https://arxiv.org/abs/2405.14867)。

**超参数调优（Hyperparameter Tuning）**：对学习率、batch size 等超参数进行细致优化，对于获得最佳蒸馏结果至关重要。不同蒸馏策略下，具体超参数的重要性也不同。建议先进行系统化的小规模实验，以确定最佳配置，再开展完整规模训练。

蒸馏过程是迭代式的，而评估在每个阶段都扮演关键角色，用于验证质量提升、泛化能力以及与预期部署条件之间的对齐程度。

## 成功指标

评估蒸馏是否成功时，可重点关注以下指标：

推理效率

- **步数缩减**：衡量 Teacher 步数与 Student 步数之间的比值。
- **实际加速比**：将 CFG 的消除考虑在内。由于 CFG 已被蒸馏进 Student，推理只需 1 次 forward pass，而不是 2×steps（因为条件/无条件各需一次）。
- **延迟**：在目标硬件上测量端到端推理时间。

输出质量

- **FID/FVD**：将 Student 和 Teacher 的输出与 ground truth 进行比较。详见 [Predict 评估](../evaluation/evaluation_predict.md) 中的指标说明。
- **控制保真度**（适用于 Transfer 模型）：测量 Blur SSIM、Canny-F1、Depth RMSE 和 Seg mIOU，以确保 Student 仍然遵循条件信号。详见 [Transfer 评估](../evaluation/evaluation_transfer.md)。
- **视觉检查**：在训练过程中定期可视化样本，对比 Student 输出、Teacher 输出和 ground truth 的不同时间采样。

## 局限性

计算需求

- **额外显存开销**：DMD2 需要同时维护多个网络（Student、Teacher、fake score network，以及可选的 discriminator），会显著提高 GPU 显存需求。可通过 FSDP、gradient checkpointing 和 gradient accumulation 等方式缓解。
- **多节点训练**：大规模蒸馏通常需要跨多个节点进行分布式训练，才能在显存受限的情况下实现足够的有效 batch size（例如 64）。

质量与速度的权衡

- **单步伪影**：极端步数压缩（例如 36→1 步）相较于多步 Teacher 输出，可能会带来细节或复杂运动上的轻微质量退化。
- **领域泛化能力**：蒸馏模型在分布外输入上的泛化能力，可能弱于 Teacher 模型。

## 案例研究

探索适用于 Cosmos 模型的蒸馏技术实际实现：

- **[蒸馏 Cosmos Transfer 1](distilling_transfer1.md)** - 使用 Knowledge Distillation（KD）和 DMD2 将 Cosmos Transfer 1 蒸馏为单步推理模型的分步指南
- **[蒸馏 Cosmos Predict 2.5](distilling_predict2.5.md)** - 展示如何通过 DMD2 蒸馏将 Cosmos Predict 2.5 Video2World 模型压缩为 4-step student model 的案例研究

