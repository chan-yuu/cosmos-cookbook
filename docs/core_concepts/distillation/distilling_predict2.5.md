# 蒸馏 Cosmos Predict 2.5

> **作者：** [Qianli Ma](https://qianlim.github.io/)
> **机构：** NVIDIA

## 概览

蒸馏过程会把一个需要较多推理步骤的预训练“Teacher”视频 diffusion model，压缩成一个能够进行少步推理的“Student”模型。两者通常共享相同的架构。其关键优势之一在于：Teacher 的 classifier-free guidance（CFG）知识会被蒸馏进 Student，因此 Student 在推理时无需再使用 CFG，并额外获得 2 倍加速。

本 cookbook 通过一个案例研究，展示我们如何使用 [DMD2 algorithm](https://arxiv.org/abs/2405.14867) 将 Cosmos Predict 2.5 Video2World 模型蒸馏为一个 4-step student model。参考代码可见[这里](https://github.com/nvidia-cosmos/cosmos-predict2.5/tree/main/cosmos_predict2/_src/predict2/distill)。

在蒸馏训练过程中，会同时训练一个辅助 critic network（在论文和代码中通常称为 “fake score net”）以及 student。训练过程会在更新 student 和 critic network 之间交替进行。
该过程包含以下步骤：

- 初始化：加载预训练的 teacher network。使用 teacher network 的权重初始化 student 和 critic network。
- 可选的监督式 warm-up：使用 teacher model 生成一个合成数据集（通常为数千组输入输出对）。这些数据可用于对 student model 进行监督式 warm-up 训练。虽然这一步在 DMD2 一类方法中较常见，但我们的实验发现，在蒸馏 4-step Text/Video2World 模型时并非必需。
- 交替训练：在 $K$ 个 critic step 和 $1$ 个 student step 之间交替。我们设定 $K=4$。student 和 critic 的训练步骤都包含各自的 loss function。注意，我们观察到加入 DMD2 论文中描述的 GAN loss 并没有明显提升，因此这里为简化起见省略了它。

## 蒸馏与常规模型训练的区别

下面是与标准 Cosmos 视频模型相比的关键代码差异：

- 蒸馏训练使用专门的 trainer 和 checkpointer（见 `cosmos_predict2/_src/predict2/distill/checkpointer/` 和 `cosmos_predict2/_src/predict2/distill/trainer/`），以同时保存和加载 student 与 critic network。
- 训练步骤（见 `cosmos_predict2/_src/predict2/distill/models/`）会在 student 更新和 critic 更新之间交替进行。student loss 也不同于标准 diffusion / flow-matching loss：我们构造了一个 distribution-matching objective，使得 student 生成的样本遵循与 teacher 相同的分布。
- 在数学形式上，我们使用了 [sCM paper](https://arxiv.org/abs/2410.11081) 中提出的 TrigFlow，作为 DMD2 与 consistency distillation（即将推出）之间共享的参数化方式。它可以与 EDM 和 RectifiedFlow 相互转换，并兼容以任一形式训练的 teacher model。

下列方面则与标准 Cosmos 模型训练保持一致：

- 数据加载流水线。
- 通过 `Conditioner` object 进行条件控制，包括 text embeddings，以及 Video2World 场景中的前几帧条件输入。
- Student 与 critic 的架构，二者通常会镜像 teacher network，并由其权重初始化。

## 快速看一眼代码

为了理解如何为你自定义的 Cosmos 模型加入 DMD2 distillation 支持，这里以 Predict 2.5 Video2World 模型为例，快速看一下代码 [[code](https://github.com/nvidia-cosmos/cosmos-predict2.5/blob/main/cosmos_predict2/_src/predict2/distill/models/video2world_model_distill_dmd2.py)]。

```python
class Video2WorldModelDistillDMD2TrigFlow(DistillationCoreMixin, TrigFlowMixin, Video2WorldModel):
    ...
```

蒸馏模型从 `DistillationCoreMixin` 继承通用的蒸馏相关代码。由于我们使用 Trigflow 作为两种蒸馏方法统一的参数化形式，`TrigFlowMixin` 提供了便捷的训练期 timestep sampling 函数。随后，该模型再继承 teacher model 类——这里是 Predict 2.5 的 `Video2WorldModel`——以复用其大部分功能，包括 tokenizer、数据处理、conditioner 等。请注意，这里的继承顺序很重要。

实现的关键在于重写 training step。负责在 student 和 critic 阶段之间交替的高层 `training_step` 位于 `DistillationCoreMixin` 中。对于 DMD2，你需要在自己的模型中实现两个方法：

Student 阶段（`training_step_generator`）：

- 冻结 critic（以及启用时的 discriminator）；解冻 student。
- 采样时间和噪声；从噪声生成少步的 student 样本。
- 将 student 生成的样本重新加噪到采样时间，然后将该 re-noised 状态分别以 cond/uncond 形式输入 teacher 以形成 CFG target；若启用了 critic，也将同一 re-noised 状态输入 critic。
- 根据 teacher 和 critic 的预测计算 DMD2 losses；仅对 student 回传梯度。若已配置，也可以选择加入 GAN 项。

Critic 阶段（`training_step_critic`）：

- 冻结 student；解冻 critic（以及启用时的 discriminator）。
- 通过一个短的 backward simulation（少量 reverse step）生成 student 样本；再重新加噪到采样时间。
- 在这些 student 样本上训练 critic 以拟合 denoising target；如果使用了 discriminator head，还要运行 real/noisy-real 路径并应用 GAN loss。

## 关键超参数说明

- `scaling`：控制时间（噪声级别）系数如何映射到 TrigFlow 参数化中。应根据 teacher model 的训练方式设置（`'edm'` 或 `'rectified_flow'`）。
- `optimizer_fake_score_config`：critic（fake score）network optimizer 的配置；其中 `lr` 字段指定 critic 的学习率。
- `student_update_freq`：控制运行 student training step 的频率。默认值为 5，表示每第 5 个训练 step 更新一次 student，其余 step 仅更新 critic。
- `tangent_warmup`：初始阶段只训练 student（不与 critic 交替）的步数。在我们的 DMD2 实验中，这个 warmup 并未带来明确收益。

## 示例训练进展

DMD2 蒸馏过程通常收敛很快。例如，在给定示例中，4-step student 在 1500 次迭代后就获得了令人满意的视频质量，这对应于 300 个 student step 和 1200 个 critic step。
![DMD2 Predict 2.5 vis 2k](../../assets/images/distillation/dmd2_predict2.5_step2k.png)

---

## 文档信息

**发布日期：** 2025 年 11 月 30 日

### 引用

如果你使用了本内容或引用了本工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_distilling_predict2_5_2025,
  title={Distilling Cosmos Predict 2.5},
  author={Ma, Qianli},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/distillation/distilling_predict2.5.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用格式：**

> Qianli Ma（2025）。蒸馏 Cosmos Predict 2.5。收录于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/distillation/distilling_predict2.5.html>

