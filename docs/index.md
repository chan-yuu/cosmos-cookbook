# Cosmos Cookbook

<div style="width: 100%; max-width: 969px; margin: 2rem 0; display: block;">
  <video autoplay loop muted playsinline style="width: 100%; max-width: 969px; height: auto; box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); display: block;">
    <source src="assets/images/homepage_video.mp4" type="video/mp4">
    您的浏览器不支持 video 标签。
  </video>
</div>

## 概览

**[NVIDIA Cosmos™](https://www.nvidia.com/en-us/ai/cosmos/)** 是一个由最先进的生成式世界基础模型（WFM）、护栏（guardrails）以及加速数据处理与整理流水线组成的平台。本 cookbook 作为 Cosmos 开放模型的实用指南，提供了用于构建、适配和部署 WFM 的分步工作流、技术配方和具体示例，帮助开发者复现成功的 Cosmos 模型部署，并根据各自领域进行定制。

Cosmos 生态系统支持完整的 Physical AI 开发生命周期——从使用预训练模型进行推理，到面向领域适配的自定义后训练。你将在这里看到以下内容：

- 快速上手的推理示例，帮助你迅速开始。
- 面向特定领域微调的高级后训练工作流。
- 经过验证、可扩展、可用于生产环境部署的配方。

## 最新更新

| **日期** | **配方** | **模型** |
|----------|------------|-----------|
| Mar 3 | [GR00T-Dreams: 面向机器人学习的合成轨迹生成](recipes/end2end/gr00t-dreams/post-training.md) | Cosmos Predict 2.5, Reason 2 |
| Feb 18 | [Cosmos Policy：面向视觉运动控制与规划的视频模型微调](recipes/post_training/predict2/cosmos_policy/post_training.md)<br><small>已升级至 Predict 2.5</small> | Cosmos Predict 2.5 |
| Feb 18 | [使用 Cosmos Reason 1 & 2 进行 3D AV Grounding 后训练](recipes/post_training/reason2/av_3d_grounding/post_training.md) | Cosmos Reason 1 & 2 |
| Feb 4 | [经典仓储环境中的工人安全](recipes/inference/reason2/worker_safety/inference.md) | Cosmos Reason 2 |
| Jan 30 | [提示词指南](getting_started/prompt_guide/reason_guide.md) | Cosmos Reason 2 |
| Jan 29 | [使用 Cosmos Reason 进行视频搜索与摘要](recipes/inference/reason2/vss/inference.md) | Cosmos Reason 2 |
| Jan 28 | [Cosmos Policy：面向视觉运动控制与规划的视频模型微调](recipes/post_training/predict2/cosmos_policy/post_training.md) | Cosmos Predict 2 |
| Jan 27 | [使用 Cosmos Reason 2 进行物理合理性预测](recipes/post_training/reason2/physical-plausibility-check/post_training.md) | Cosmos Reason 2 |
| Jan 26 | [使用 Cosmos Reason 2 进行智能交通后训练](recipes/post_training/reason2/intelligent-transportation/post_training.md) | Cosmos Reason 2 |

## 即将举行的活动

### NVIDIA GTC 2026

欢迎注册将于 **2026 年 3 月 16–19 日** 举办的 [NVIDIA GTC](https://www.nvidia.com/gtc/)，并将 [Cosmos 相关会议](https://www.nvidia.com/gtc/session-catalog/?sessions=S81667,CWES81669,DLIT81644,DLIT81698,S81836,S81488,S81834,DLIT81774,CWES81733,CWES81568) 加入你的日历。不要错过 CEO Jensen Huang 将于 3 月 16 日（周一）太平洋时间上午 11:00 在 SAP Center 带来的必看主题演讲。

### NVIDIA Cosmos Cookoff

隆重推出 **[NVIDIA Cosmos Cookoff](https://luma.com/nvidia-cosmos-cookoff)** —— 一个为期四周的线上 Physical AI 挑战赛，面向机器人、AV 和 Vision AI 开发者，于 **1 月 29 日至 2 月 26 日** 举行。

使用 NVIDIA Cosmos Reason 和 Cosmos Cookbook 中的配方进行构建——从第一视角机器人推理到物理合理性检查，再到具备交通感知能力的模型——即有机会赢得 **5,000 美元**、**NVIDIA DGX Spark** 等奖项！

**[立即注册 →](https://luma.com/nvidia-cosmos-cookoff)**

由 Nebius 和 Milestone 赞助。

## 开源社区平台

Cosmos Cookbook 是一个开源资源平台，NVIDIA 与更广泛的 Physical AI 社区可在此共享实用工作流、成熟技术以及特定领域的适配经验。

**📂 仓库：** [https://github.com/nvidia-cosmos/cosmos-cookbook](https://github.com/nvidia-cosmos/cosmos-cookbook)

我们欢迎各种贡献——从新的示例和工作流改进，到 bug 修复和文档更新。大家可以共同演进最佳实践，加速 Cosmos 模型在各个领域中的采用。

**📊 Physical AI 数据集：** 可在 Hugging Face 上的 [NVIDIA Physical AI Collection](https://huggingface.co/collections/nvidia/physical-ai) 获取面向自动驾驶、智能交通系统、机器人、智能空间和仓储环境的精选数据集。

<a id="case-study-recipes"></a>
## 案例配方

Cosmos Cookbook 包含全面的用例，展示 Cosmos 平台在现实世界中的实际应用。

### [**Cosmos Predict**](https://github.com/nvidia-cosmos/cosmos-predict2.5)

#### 未来状态预测与生成

| **工作流** | **说明** | **链接** |
|--------------|-----------------|----------|
| **推理** | 面向智能交通系统的 Text2Image 合成数据生成 | [ITS 合成数据生成](recipes/inference/predict2/inference-its/inference.md) |
| **训练** | 通过潜空间帧注入为机器人操作进行 Cosmos Predict 2 微调，以实现视觉运动控制 | [Cosmos Policy](recipes/post_training/predict2/cosmos_policy/post_training.md) |
| **训练** | 提升真实感和提示词对齐度的交通异常生成 | [交通异常生成](recipes/post_training/predict2/its-accident/post_training.md) |
| **训练** | 面向人形机器人学习的合成轨迹数据生成 | [GR00T-Dreams](recipes/post_training/predict2/gr00t-dreams/post-training.md) |
| **训练** | 用于体育视频生成的 LoRA 后训练，提升球员动态和规则一致性 | [体育视频生成](recipes/post_training/predict2_5/sports/post_training.md) |

> **高级主题：** 参考 [蒸馏 Cosmos Predict 2.5](core_concepts/distillation/distilling_predict2.5.md)，了解如何使用 DMD2 将模型蒸馏为一个 4-step student。

### [**Cosmos Transfer**](https://github.com/nvidia-cosmos/cosmos-transfer2.5)

#### 多控制视频生成与增强

| **工作流** | **说明** | **链接** |
|--------------|-----------------|----------|
| **指南** | 掌握使用 Edge、Depth、Segmentation 和 Vis 模态对视频生成进行精确控制的方法，以实现结构保留和语义替换 | [控制模态指南](core_concepts/control_modalities/overview.md) |
| **推理** | 使用图像参考并结合 edge/depth/segmentation 控制的风格引导视频生成 | [风格引导生成](recipes/inference/transfer2_5/inference-image-prompt/inference.md) |
| **推理** | 面向交通异常场景的 CARLA simulator-to-real 增强 | [CARLA Sim2Real](recipes/inference/transfer2_5/inference-carla-sdg-augmentation/inference.md) |
| **推理** | 用于背景替换、光照调整和对象变换的多控制视频编辑 | [真实世界视频编辑](recipes/inference/transfer2_5/inference-real-augmentation/inference.md) |
| **推理** | 面向稀缺生物数据集、使用基于 edge 控制与 FiftyOne 的领域迁移流水线 | [BioTrove 飞蛾增强](recipes/inference/transfer2_5/biotrove_augmentation/inference.md) |
| **推理** | 使用多模态控制的仿真数据天气增强流水线 | [天气增强](recipes/inference/transfer1/inference-its-weather-augmentation/inference.md) |
| **推理** | 面向多视角仓储环境的 CG-to-real 转换 | [仓储仿真](recipes/inference/transfer1/inference-warehouse-mv/inference.md) |
| **推理** | 面向机器人导航任务的 Sim2Real 数据增强 | [X-Mobility Navigation](recipes/inference/transfer1/inference-x-mobility/inference.md) |
| **推理** | 面向人形机器人的合成操作动作生成 | [GR00T-Mimic](recipes/inference/transfer1/gr00t-mimic/inference.md) |
| **训练** | 使用世界场景地图进行空间条件多视角 AV 视频生成的 ControlNet 后训练 | [多视角 AV 生成](recipes/post_training/transfer2_5/av_world_scenario_maps/post_training.md) |

### [**Cosmos Reason**](https://github.com/nvidia-cosmos/cosmos-reason1)

#### 视觉语言推理与质量控制

| **工作流** | **说明**                                                           | **链接**                                                                                                |
| ------------ | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **指南** | 面向 Cosmos Reason 2 的全面提示词指南，涵盖消息结构、采样参数和领域模式 | [Cosmos Reason 2 提示词指南](getting_started/prompt_guide/reason_guide.md)                   |
| **推理** | 面向大规模视频摘要、问答和直播流告警的 GPU 加速视频分析流水线 | [视频搜索与摘要](recipes/inference/reason2/vss/inference.md)                   |
| **推理** | 工业仓储环境中的零样本安全合规与危险检测 | [经典仓储环境中的工人安全](recipes/inference/reason2/worker_safety/inference.md) |
| **推理** | 面向社交机器人的第一视角社交与物理推理             | [第一视角社交推理](recipes/inference/reason2/intbot_showcase/inference.md) |
| **训练** | 使用 Cosmos Reason 1 & 2 进行自动驾驶中的 3D 车辆定位 | [3D AV Grounding（Reason 1 & 2）](recipes/post_training/reason2/av_3d_grounding/post_training.md) |
| **训练** | 使用生产数据对 Cosmos Reason 2 进行后训练，用于 AV 视频描述与 VQA | [AV Video Caption VQA（Reason 2）](recipes/post_training/reason2/video_caption_vqa/post_training.md)     |
| **训练** | 使用 WTS 数据对 Cosmos Reason 2 进行后训练，用于智能交通场景理解 | [智能交通（Reason 2）](recipes/post_training/reason2/intelligent-transportation/post_training.md) |
| **训练** | 使用 Cosmos Reason 2 进行视频质量评估的物理合理性预测 | [物理合理性（Reason 2）](recipes/post_training/reason2/physical-plausibility-check/post_training.md) |
| **训练** | 用于视频质量评估的物理合理性检查                  | [物理合理性（Reason 1）](recipes/post_training/reason1/physical-plausibility-check/post_training.md)             |
| **训练** | 面向仓储环境的 Spatial AI 理解                       | [Spatial AI Warehouse](recipes/post_training/reason1/spatial-ai-warehouse/post_training.md)             |
| **训练** | 智能交通场景理解与分析               | [智能交通（Reason 1）](recipes/post_training/reason1/intelligent-transportation/post_training.md) |
| **训练** | 面向自动驾驶的 AV 视频描述与视觉问答 | [AV Video Caption VQA（Reason 1）](recipes/post_training/reason1/av_video_caption_vqa/post_training.md)  |
| **训练** | 用于 MimicGen 机器人学习数据生成的时间定位         | [时间定位](recipes/post_training/reason1/temporal_localization/post_training.md)           |
| **训练** | 在 WM-811k 上进行监督微调的晶圆图异常分类  | [晶圆图分类](recipes/post_training/reason1/wafermap_classification/post_training.md)      |

### [**Cosmos Curator**](https://github.com/nvidia-cosmos/cosmos-curate)

| **工作流** | **说明**                                      | **链接**                                                                        |
| ------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------- |
| **数据整理** | 为 Cosmos Predict 2 后训练整理视频数据 | [Predict 2 数据整理](recipes/data_curation/predict2_data/data_curation.md) |
| **分析** | 使用 Time Series K-Means 对嵌入轨迹进行高级视频聚类 | [使用 Time Series K-Means 进行视频聚类](recipes/data_curation/embedding_analysis/embedding_analysis.md) |

### **端到端工作流**

| **工作流** | **说明** | **链接** |
|--------------|-----------------|----------|
| **GR00T-Dreams** | 用于合成机器人轨迹生成的端到端流水线：在 GR1 数据上对 Cosmos Predict 2.5 进行后训练、生成轨迹，并使用 Cosmos Reason 2 作为视频评论器执行拒绝采样 | [GR00T-Dreams](recipes/end2end/gr00t-dreams/post-training.md) |
| **SDG Pipeline** | 使用 CARLA、Cosmos Transfer 2.5 和 Cosmos Reason 1 构建交通场景完整合成数据生成流水线 | [智慧城市 SDG](recipes/end2end/smart_city_sdg/workflow_e2e.md) |

## 面向 Physical AI 的 Cosmos 模型

Cosmos 开放模型家族由五个核心仓库组成，每个仓库都面向 AI 开发工作流中的特定能力：

**[Cosmos Curator](https://github.com/nvidia-cosmos/cosmos-curate)** - 一个基于 Ray 的 GPU 加速视频整理流水线。支持多模型分析、内容过滤、标注和去重，可用于推理与训练数据准备。

**[Cosmos Predict](https://github.com/nvidia-cosmos/cosmos-predict2.5)** - 一个用于未来状态预测的 diffusion transformer，提供 text-to-image 和 video-to-world 生成能力，并为机器人和仿真提供专用变体。支持针对特定领域预测任务的自定义训练。

**[Cosmos Transfer](https://github.com/nvidia-cosmos/cosmos-transfer2.5)** - 一个具备 ControlNet 和 MultiControlNet 条件控制（包括 depth、segmentation、LiDAR 和 HDMap）的多控制视频生成系统。包含 4K 超分能力，并支持自定义控制模态与领域适配训练。

**[Cosmos Reason](https://github.com/nvidia-cosmos/cosmos-reason1)** - 一个 7B 视觉语言模型，用于物理世界扎根推理。可处理空间/时间理解和 chain-of-thought 任务，并支持面向 embodied AI 应用和领域特定推理的微调。

**[Cosmos RL](https://github.com/nvidia-cosmos/cosmos-rl)** - 一个分布式训练框架，同时支持监督微调（SFT）和强化学习方法，具备弹性策略 rollout、FP8/FP4 精度支持，以及面向大规模 VLM 和 LLM 训练的优化。

所有模型都包含预训练 checkpoint，并支持面向特定领域适配的自定义训练。下图展示了各组件在推理与训练工作流中的交互方式。

![Cosmos Overview](assets/images/cosmos_overview.png)

## ML/Gen AI 概念

本 cookbook 围绕横跨（可控）**推理**与**训练**使用场景的关键概念组织：

**1. [提示词指南](getting_started/prompt_guide/overview.md)** - 学习适用于 Cosmos 模型的高效提示词策略。内容涵盖消息结构、媒体顺序、采样参数和领域模式，帮助你从 Cosmos Reason 与其他视觉语言模型中获得最佳结果。

**2. [控制模态](core_concepts/control_modalities/overview.md)** - 掌握如何在 Cosmos Transfer 2.5 中使用 Edge、Depth、Segmentation 和 Vis 模态精确控制视频生成。内容涵盖结构保留、语义替换、光照一致性，以及实现高保真、可控视频变换的多控制方法。

**3. [数据整理](core_concepts/data_curation/overview.md)** - 使用 Cosmos Curator 通过模块化、可扩展的处理流水线准备你的数据集。包括切分、字幕生成、过滤、去重、任务特定采样，以及云原生或本地执行。

**4. [模型后训练](core_concepts/post_training/overview.md)** - 使用你整理好的数据对基础模型进行微调。内容涵盖 Predict（2 和 2.5）、Transfer（1 和 2.5）以及 Reason 1 的领域适配，监督微调、LoRA 或强化学习的配置，以及利用 Cosmos RL 进行大规模分布式 rollout。

**5. [评估与质量控制](core_concepts/evaluation/overview.md)** - 通过指标、可视化和定性检查，确保后训练模型具备良好的对齐性和稳健性。你还可以利用 Cosmos Reason 1 作为质量过滤器（例如用于合成数据拒绝采样）。

**6. [模型蒸馏](core_concepts/distillation/overview.md)** - 在保留性能的同时，将大型基础模型压缩为更小、更高效的变体。内容包括适用于 Cosmos 模型的知识蒸馏技术、teacher-student 训练配置，以及面向边缘设备和资源受限环境的部署优化。

## 示例画廊

展示 Cosmos Transfer 在 Physical AI 各领域中的可视化结果示例：

- **[机器人领域适配](gallery/robotics_inference.md)** - 面向机器人操作的 sim-to-real 迁移，涵盖多样材质、光照和环境。
- **[自动驾驶领域适配](gallery/av_inference.md)** - 面向驾驶场景的多控制视频生成，覆盖不同天气、光照和昼夜条件。

## 快速开始路径

本 cookbook 为 **推理** 和 **训练** 工作流都提供了灵活的切入点。每个部分都包含可运行脚本、技术配方和完整示例。

- **推理工作流：** [快速开始](getting_started/setup.md)，用于环境设置和即时模型部署
- **云端部署：** [云平台](getting_started/cloud_platform.md)，用于在 Nebius、Brev 等平台上一键启动云实例
- **Physical AI 数据集：** Hugging Face 上的 [NVIDIA Physical AI Collection](https://huggingface.co/collections/nvidia/physical-ai)，提供跨领域精选数据集
- **数据处理：** [数据处理与分析](core_concepts/data_curation/overview.md)，用于内容分析工作流
- **训练工作流：** [模型训练与微调](core_concepts/post_training/overview.md)，用于领域适配
- **案例配方：** [案例配方](#case-study-recipes)，按应用领域组织
