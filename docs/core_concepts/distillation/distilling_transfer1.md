# 蒸馏 Cosmos Transfer 1 模型

> **作者：** [Grace Lam](https://www.linkedin.com/in/grace-lam/)
> **机构：** NVIDIA

## 来自 Cosmos Transfer 1 仓库的说明

- [蒸馏 Cosmos Transfer 1-7B [Depth | Edge | Keypoint | Segmentation | Vis]](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/distillation_cosmos_transfer1_7b.md) **[支持多 GPU]**

## 案例研究：蒸馏 Cosmos Transfer 1 Edge

本教程展示了一个关于单步蒸馏 36-step Cosmos Transfer 1-7B Edge 模型的案例。原始模型由于使用 classifier-free guidance（CFG），总共需要 72 次推理（36 steps x 2）；而蒸馏后的模型无需 CFG，只需单次推理即可完成。这在保持输出质量的同时实现了 72 倍加速。

### 概览

我们的 recipe 是一个两阶段蒸馏流水线。

#### 第 1 阶段：Knowledge Distillation（KD）

- 我们使用 teacher model 生成了一个包含 10,000 组 noise-video 对的合成数据集，用于 Knowledge Distillation。
- 强有力的 warmup 阶段被证明对后续 DMD2 的成功至关重要，因为这种合成数据方案明显优于另一种使用真实数据进行 L2 回归 warmup 的替代方法。
- 我们以 1e-5 的学习率、64 的 global batch size 训练 KD 阶段，共 10,000 次迭代。

#### 第 2 阶段：Improved Distribution Matching Distillation（DMD2）

- 我们使用了 [DMD2](https://arxiv.org/abs/2405.14867)，这是一种当前先进的基于分布的蒸馏方法，结合了对抗蒸馏与 variational score distillation。
- 主要挑战在于同时维护多份网络副本带来的内存约束（student model、teacher model、fake score network 和 discriminator）。我们通过 FSDP、CP8、gradient checkpointing 和 gradient accumulation 解决了这一问题，从而在 16 个节点上实现了等效 batch size 为 64 的训练。
- 我们以 5e-7 的学习率、5 的 guidance scale、1e-3 的 GAN loss 权重、5 的 student update frequency，以及 64 的 global batch size 训练 DMD2 阶段，共 24,000 次迭代。

### Knowledge Distillation（KD）

#### 数据集

KD 通过最小化 Student 模型单步生成与 Teacher 模型多步生成（对 Cosmos Transfer 1 来说为 36 步）之间的回归损失来进行训练。因此，KD 需要一个预先的数据生成阶段，以创建由 Teacher 模型输入输出对构成的合成数据集。对于 Cosmos Transfer 1 Edge，输入包括随机噪声、文本提示词和 canny edge map，输出则是生成的视频。

在合成数据生成过程中，我们从用于 Cosmos Transfer 1 Edge 的原始训练数据集中随机采样了 10,000 个样本。我们提取这些样本中的文本提示词和 canny 输入，并将其送入 Teacher 模型（Cosmos Transfer 1 Edge）生成对应输出视频。最终得到的 KD 数据集保留了原始文本提示词和 canny 输入，同时额外存储随机噪声张量和 Teacher 生成的视频，从而为 Student 模型训练构建完整的输入输出对。

为了确保蒸馏训练所需的合成数据质量足够高，我们使用最佳推理超参数生成 Teacher 输出，包括 guidance scale 为 7，并配合 negative prompting。

#### 超参数

我们早期的蒸馏实验表明，batch size 是实现有效知识迁移的关键因素。起初，batch size 为 8 的小规模实验表现出较为有限的蒸馏质量；而扩展到 32 及以上后，效果明显改善。基于这些发现，我们在所有完整规模的蒸馏实验中都采用了 64 的 batch size。

在 KD 超参数优化中，我们系统性地扫描了学习率，并确定 1e-5 在我们的实验配置下表现最佳。

#### 可视化

在蒸馏过程中，我们持续记录模型输出样本，以定期监控训练进展。这一监控功能在 Cosmos Transfer 1 的蒸馏代码库中以 training callback 的形式实现，可用于系统化评估。

可视化布局纵向展示四个关键组成部分：Student 1-step sample、Teacher 1-step sample、canny edge 输入，以及 ground-truth reference video。每一行展示对应视频片段的三个时间采样——即首帧、中间帧和末帧——从而能够直接比较不同训练迭代下的蒸馏质量与时间一致性。

下面展示了蒸馏训练过程中的代表性可视化结果。

约 5k steps 后：

![KD vis 5k](../../assets/images/distillation/kd_transfer1_step5k.jpg)

约 10k steps 后：

![KD vis 10k](../../assets/images/distillation/kd_transfer1_step10k.jpg)

### Improved Distribution Matching Distillation（DMD2）

#### 数据集

DMD2 通过对抗训练结合 variational score distillation，优化 Student 模型以匹配 Teacher 模型的输出分布。与 Knowledge Distillation 不同，DMD2 需要的是多样化的真实 ground-truth 视频数据集，而不是 Teacher 生成的合成输出。我们使用了用于 Cosmos Transfer 1 Edge 的原始训练数据集。

#### 超参数

与 Knowledge Distillation 类似，batch size 和学习率依然是关键因素。最佳性能出现在 batch size 为 64、学习率为 5e-7 时。DMD2 还引入了额外的超参数。我们设定每次 generator 更新前进行 4 次 discriminator 和 fake score network 更新（`student_update_freq=5`）。在 guidance scale 的优化中，我们发现取值 5 能在输出饱和度和细节锐度之间取得最佳平衡。对于损失加权，我们使用 0.001 的 GAN loss 权重，有效平衡了对抗目标与 score distillation loss。

#### 可视化

在蒸馏过程中，我们持续记录模型输出样本，以定期监控训练进展。这一监控功能在 Cosmos Transfer 1 的蒸馏代码库中以 training callback 的形式实现，可用于系统化评估。

可视化布局纵向展示四个关键组成部分：Student 1-step sample、Teacher 1-step sample、canny edge 输入，以及 ground-truth reference video。每一行展示对应视频片段的三个时间采样——即首帧、中间帧和末帧——从而能够直接比较不同训练迭代下的蒸馏质量与时间一致性。

下面展示了蒸馏训练过程中的代表性可视化结果。

约 10k steps 后：

![DMD2 vis 10k](../../assets/images/distillation/dmd2_transfer1_step10k.jpg)

约 20k steps 后：

![DMD2 vis 20k](../../assets/images/distillation/dmd2_transfer1_step20k.jpg)

---

## 文档信息

**发布日期：** 2025 年 10 月 9 日

### 引用

如果你使用了本内容或引用了本工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_distilling_transfer1_2025,
  title={Distilling Cosmos Transfer 1 Models},
  author={Lam, Grace},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/distillation/distilling_transfer1.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用格式：**

> Grace Lam（2025）。蒸馏 Cosmos Transfer 1 模型。收录于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/distillation/distilling_transfer1.html>

