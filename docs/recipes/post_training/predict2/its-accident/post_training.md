# 使用 Cosmos Predict2 生成交通异常

> **作者：** [Arslan Ali](https://www.linkedin.com/in/arslan-ali-ph-d-5b314239/), [Grace Lam](https://www.linkedin.com/in/grace-lam/), [Amol Fasale](https://www.linkedin.com/in/amolfasale/), [Jingyi Jin](https://www.linkedin.com/in/jingyi-jin/)
> **机构：** NVIDIA

## 概述

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Predict 2](https://github.com/nvidia-cosmos/cosmos-predict2) | 后训练 | 交通异常生成，具有更好的真实感和 prompt 对齐能力 |

在 Intelligent Transportation Systems (ITS) 中，收集交通事故、行人横穿、路口阻塞等稀有事件的真实世界数据面临诸多挑战：

- **隐私问题**：记录和使用真实事故画面会带来伦理和法律问题。
- **发生频率低**：关键安全事件天生罕见，使数据收集成本高且耗时。
- **标注成本高**：对交通事件进行专家标注需要专业知识。
- **安全风险**：为了采集数据而布置真实事故既危险又不切实际。

Synthetic data generation (SDG) 提供了一种实用途径来扩充现有数据集，使团队能够在保持对场景参数和数据质量控制的同时，大规模创建有针对性的场景。

## 挑战：ITS 中的稀有事件数据

对预训练 Cosmos Predict 2 模型的初步评估显示，在生成车辆碰撞场景时存在一些不足：

- 不真实的运动动力学
- 车辆尺寸过大（很可能是预训练中 dash cam 偏置导致）
- 缺乏事件特定行为
- 难以维持固定摄像机视角

尽管预训练模型在常规交通场景中表现出色，但在 ITS 特定 prompts 下测试碰撞场景时，它表现不佳。这证实了需要使用富含异常的数据进行有针对性的后训练，尤其是包含固定 CCTV 视角事故画面的数据。

## 前置条件

在开始训练之前：

1. **环境设置**：按照 [Setup guide](https://github.com/nvidia-cosmos/cosmos-predict2/blob/main/docs/setup.md) 中的安装说明完成环境配置。
2. **模型 checkpoints**：按照 [Setup guide](https://github.com/nvidia-cosmos/cosmos-predict2/blob/main/docs/setup.md) 中 *Downloading Checkpoints* 一节下载所需模型权重。

### 关键依赖

| Package | Version | Purpose |
|---------|---------|---------|
| `torch` | >=2.5.0 | 深度学习框架 |
| `megatron-core` | >=0.11.0 | 分布式训练工具 |
| `transformers` | >=4.45.0 | Hugging Face 模型加载 |
| `peft` | >=0.13.0 | LoRA 实现 |
| `einops` | >=0.8.0 | Tensor 操作 |
| `hydra-core` | >=1.3.0 | 配置管理 |

## 我们的方法：基于 LoRA 的领域适配

本案例研究记录了一个详细的后训练工作流，使用 Cosmos Predict 2 Video2World 与 **Low-Rank Adaptation (LoRA)**，重点提升模型从固定 CCTV 视角生成交通异常视频的能力。我们不对整个模型进行微调，而是采用 LoRA，以高效方式将预训练 foundation model 适配到 ITS 特定需求。

### 为什么 LoRA 适合 ITS 应用？

LoRA (Low-Rank Adaptation) 特别适合 ITS 领域适配，原因如下：

#### 关键优势：面向稀有事件的数据效率

与拥有数百万样本的通用视频数据集不同，交通事故数据集通常只有数百到数千个示例。这种数据稀缺性使 LoRA 成为最佳选择：

- **在有限数据下依然有效**：LoRA 仅用 1,000-2,000 个训练样本就能实现有意义的适配。
- **降低过拟合风险**：参数更少（45M 对比 2B）意味着更不容易记忆有限训练数据。
- **更好的泛化能力**：受限的参数空间迫使模型学习可泛化的模式，而不是具体示例。
- **利用预训练能力**：LoRA 建立在基础模型已有知识之上，只需少量事故特定数据即可完成适配。

在我们的案例研究中，尽管 clips 非常有限，LoRA 依然实现了成功适配，而完整微调很可能失败或发生严重过拟合。

#### 参数效率

- **极小存储开销**：LoRA 只在一个 2B 参数模型上增加约 45M 个可训练参数（≈2% 增量）。
- **快速部署**：LoRA adapters 相比完整模型 checkpoints（5-50GB）体积很小（10-100MB）。
- **多领域支持**：不同交通场景（高速公路、路口、停车场）可以使用不同的 LoRA adapter。

#### 资源优化

- **更短训练时间**：训练一个 2B 模型使用 LoRA 只需 1-2 小时，而完整微调需 2-4 小时。
- **更低 GPU 显存需求**：LoRA 需要 20GB GPU 显存，而完整模型训练需要 50GB。
- **更快迭代**：可以快速尝试不同训练配置。

#### 保留基础能力

- **无灾难性遗忘**：基础模型的通用视频生成能力保持完好
- **增量学习**：在不降低通用性能的前提下增加 ITS 特定知识
- **可回退选项**：需要时可禁用 LoRA，以使用原始模型行为

### LoRA 配置

基于 [LoRA paper (Hu et al., 2021)](https://arxiv.org/abs/2106.09685)，我们的配置如下：

- **Target Modules**：`q_proj`, `k_proj`, `v_proj`, `output_proj`, `mlp.layer1`, `mlp.layer2`
- **Rank**：16（该值决定低秩分解的维度——更高的 rank 表达能力更强，但参数也更多。）
- **Alpha**：16（该缩放超参数控制 LoRA 更新幅度——通常设为与 rank 相同，以获得平衡学习。）
- **Training Data**：正常交通场景与事故场景按 1:1 混合。

该配置聚焦于注意力机制和前馈层，这对于以下条件至关重要：

- 理解车辆之间的空间关系。
- 捕捉碰撞的时间动态。
- 维持一致的摄像机视角。
- 生成物理上合理的运动。

## 数据准备

为了解决模型局限性，我们开发了一条多源数据流水线，结合了以下数据：

- ITS 正常交通场景：100 小时来自不同路口、不同时间段的交通监控视频，全部由固定 CCTV 视角采集（无 dash cam 或移动相机视角）
- ITS 事故场景：来自不同路口、不同时间段的事故场景汇编，同样全部由固定 CCTV 视角采集（总计约 3.5 小时视频）。

> **免责声明**：本案例研究中收集的所有数据仅用于研究概念验证和演示目的。这些数据未并入预训练数据集。本示例仅用于说明数据整理方法和后训练工作流。

### 切分与描述生成

**ITS 事故场景**：原始的 5-10 分钟汇编视频使用 `cosmos-curate` 结合 `transnetv2` 场景检测和客观描述生成，被切分为独立 clips。

