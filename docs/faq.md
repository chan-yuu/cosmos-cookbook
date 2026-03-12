# 常见问题解答

本文档汇总了来自多个来源的完整 FAQ 信息。

## 常规问题

### 什么是 NVIDIA Cosmos？

**[NVIDIA Cosmos™](https://www.nvidia.com/en-us/ai/cosmos/)** 是一个用于推进 Physical AI 的世界基础模型（WFM）开发平台。其核心是 Cosmos WFM——公开提供的预训练多模态模型，开发者可以开箱即用地用它们生成以视频形式呈现的世界状态并进行 Physical AI 推理，或通过后训练开发专用的 Physical AI 模型。NVIDIA Cosmos 还包括先进的 tokenizer、guardrails、加速数据处理流水线以及后训练脚本。

### Cosmos 的主要组成部分是什么？

#### Cosmos 世界基础模型（WFM）

Cosmos 世界基础模型（WFM）是用于虚拟世界生成、以推动 Physical AI 发展的预训练生成式 AI 模型。WFM 家族包括：

- 用于生成未来世界状态视频的 **Cosmos Predict**
- 用于条件式合成数据生成的 **Cosmos Transfer**
- 用于 Physical AI 推理的 **Cosmos Reason**

这些模型都可完全定制，用于开发专门的 Physical AI 模型。

#### Cosmos Curator

一个 GPU 加速的视频预处理与整理工具包，用于准备高质量数据集。

#### Cosmos Tokenizer

用于高效地将视觉数据转换为 token。

### Cosmos 是为哪些人设计的？

Cosmos 面向以下领域的开发者和 ISV：

- 机器人
- 自动驾驶汽车
- 仿真
- 计算机视觉应用

### Cosmos 平台有哪些技术能力？

Cosmos 平台提供以下能力：

- 可立即部署的预训练世界基础模型（WFM）
- GPU 加速的数据处理与整理工具
- 面向特定领域适配的后训练框架
- CUDA 优化的推理与训练流水线
- 用于 Physical AI 模型训练的合成数据生成

## 模型

### Cosmos 的主要使用场景是什么？

**数据整理：** Cosmos 平台包含用于视频数据及视频数据搜索的 Cosmos Curator，可帮助处理海量真实或合成数据的开发者加速数据整理，以训练 Physical AI 模型。

**加速合成数据生成（SDG）：** Cosmos WFM 专为从多个维度加速 SDG 而设计。

- 借助 **Cosmos Predict**，开发者可以从文本提示词或一对图像生成合成数据。输出包括预测的下一帧或插值帧——非常适合边缘场景，或从单个输入探索多个场景。
- **Omniverse** 可创建逼真的 3D 场景，作为 **Cosmos Transfer** 的输入（也称为 “ground truth”），后者会在不同环境和光照条件下对其进行扩增。该过程可生成逼真、可扩展、增强后的数据，用于机器人和自动驾驶训练以及计算机视觉应用。
- **Cosmos Reason** 可作为合成数据的评论器（critic）。它会根据视频输入与文本提示词的匹配程度进行打分，并生成描述以帮助整理训练数据。
- 这些模型的任意组合都可以加速合成数据生成流程。结合 [NVIDIA Isaac Sim](https://developer.nvidia.com/isaac/sim)、[AV Simulation](https://www.nvidia.com/en-us/use-cases/autonomous-vehicle-simulation/)、[Isaac GR00T](https://developer.nvidia.com/isaac/gr00t)，这些模型可以解锁多种 SDG 工作流。

**后训练：** Cosmos WFM 完全可定制，可开发针对客户数据量身定制的下游视觉、机器人或自动驾驶基础模型。后训练可用于改变输出类型、输出数量、输出质量、输出风格或输出视角。

### 在哪里可以找到用于训练 Physical AI 模型的数据集？

NVIDIA 在 Hugging Face 的 [NVIDIA Physical AI Collection](https://huggingface.co/collections/nvidia/physical-ai) 中提供经过整理的、开放的、商业级 Physical AI 开发数据集。该集合包括以下数据集：

- **自动驾驶汽车**：驾驶场景、合成数据和遥操作数据集
- **机器人**：GR00T、操作、抓取和导航数据集
- **智能空间和仓储**：多摄像头跟踪、检测和空间智能数据集
- **领域特定训练与评估**：适用于各种 Physical AI 应用的专用数据集

这些数据集专为与 Cosmos 模型无缝协同工作而设计，也可作为面向特定领域后训练工作流的起点。

### Cosmos 模型与其他视频基础模型有何不同？

Cosmos 世界基础模型专为 Physical AI 应用而设计。这些模型公开可用并且可定制，其中 Cosmos Predict 和 Cosmos Reason 支持针对自动驾驶、机器人以及视觉动作生成模型进行后训练。

### 什么是 Policy Initialization？

Policy Initialization 是通过修改世界基础模型（WFM）的输出头，从中开发策略模型（policy model）的过程。策略模型将观测状态（例如视频）映射为动作。它可以通过为 Cosmos 世界基础模型添加适用于动作选择的新输出头并进行后训练来初始化（从 video head → action head）。

### 什么是 Policy Evaluation？

Policy Evaluation 是评估已训练策略模型的过程。它可以通过对特定输入（例如动作或指令）进行条件控制/播种（seeding），并分析模型输出完成。该步骤可确保模型能够正确地将状态映射为动作，并在真实世界或仿真环境中按预期表现。

### Cosmos 能否用于创意内容生成？

Cosmos 模型可以在 NVIDIA Open Model License 下生成视频内容，但该平台主要面向 Physical AI 应用，而非创意内容生成。

### 什么是 multiverse simulation？

Multiverse simulation 指从给定状态生成多个未来结果。Cosmos 与 NVIDIA Omniverse 集成后，可以为预测性维护和自主决策等任务模拟多种场景。

### Cosmos Reason 是 VLM、VLA，还是 MLLM？

Cosmos Reason 1 是一个物理推理引擎，旨在通过自然语言解释来分析真实世界场景。它可以作为 Vision Language Model（VLM）或 Multi-Modal Large Language Model（MLLM）运行，并内置 chain-of-thought 推理。与将感知输入映射为可执行动作的 Vision-Language-Action（VLA）模型不同，Cosmos Reason 1 使用面向空间、时间和物理的分层本体，生成关于安全性、因果关系和物体交互的文本推理轨迹。

### Cosmos Reason 输出什么？

当 VLA 输出电机控制命令时，Cosmos Reason 1 输出的是诸如 “坡度超过车辆倾斜容差” 这样的文本洞见，这些内容还需要通过转换层才能用于机器人执行。该模型可以进行高层规划和可解释的安全检查，但不能直接控制执行器或在动态环境中导航。

## 技术细节

### 什么是 3D consistency，如何测试？

3D consistency 衡量模型在 3D 场景中维持空间对齐的能力。Cosmos 的测试方法如下：

**测试设置**：从 500 个整理后的视频中选取静态场景，并使用以下指标：

- 几何一致性（例如 Sampson error、姿态估计）
- 视图合成一致性（PSNR、SSIM、LPIPS）

**关键指标**：

- **Sampson error**：值越低表示几何精度越高
- **姿态估计成功率**：百分比越高表示相机对齐效果越好
- **PSNR 和 SSIM**：分数越高表示合成视图质量越高
- **LPIPS**：值越低表示感知相似度越好

### 什么是 physics alignment，如何评估？

Physics alignment 用于测试模型模拟重力、碰撞等物理动态的能力。

**评估方法**：在虚拟环境中的受控场景下，使用多个指标进行评估

**关键指标**：

- **PSNR**：值越高表示噪声越少、像素精度越高
- **SSIM**：值越高表示视觉保真度越好
- **DreamSim**：评估对象和运动的语义一致性
- **IoU**：衡量预测对象区域与真实对象区域的重叠程度，以评估其是否符合物理预期

### Cosmos 世界基础模型使用什么精度策略？

**训练策略**：混合精度方法

- 同时维护 FP32 和 BF16 的权重副本
- 梯度仅以 BF16 计算
- 最终存储和推理使用 BF16

**当前限制**：

- 这些仓库当前尚不支持 FP8
- FP8 和 FP4 训练能力仍在开发中

### 后训练的基础设施要求

#### 最低配置

**对于 Cosmos Reason 1-7B：**

- **SFT 训练**：至少需要 2x 80GB GPU
- **RL 训练**：至少需要 4x 80GB GPU

**通用要求**：

- 具备足够显存的 NVIDIA GPU
- CUDA toolkit 兼容性
- 用于分布式训练的高速互连

#### 分布式训练要求

**网络：**

- **推荐**：使用 InfiniBand 或 RoCE 以实现高效通信
- **支持**：AWS EFA
- **必要条件**：多 GPU 配置需要高带宽、低延迟连接

### 优化策略

**流水线优化：**

- 基于 Ray 的流水线允许指定 GPU 类型。
- 支持动态硬件检测。
- 可在不同流水线阶段利用混合 GPU 类型。
- 遥测可确保高效利用资源。

**内存管理：**

- 内存需求因模型大小和数据集特征而异。
- 提示词长度和 chain-of-thought 长度会影响内存需求。
- 支持面向大规模部署的水平扩展。

### 性能特征

#### 压缩与质量

**视频压缩的影响：**

- 更高的压缩率可能影响生成质量。
- 使用更少 token 的时序压缩可能降低质量。
- 最佳设置取决于具体应用需求。
- 建议通过测试寻找最佳权衡。

#### 处理性能

**Cosmos Curator 性能：**

- 相比基于 CPU 的流水线，提供 GPU 加速处理。
- 针对大规模视频处理工作负载进行了优化。

**Tokenizer 性能：**

- 针对视频数据进行了优化压缩与处理
- 同时支持训练和推理工作负载。

### 模型架构细节

#### 训练配置

**层管理：**

- 在 SFT 和 RL 训练过程中不会冻结任何层。
- 采用全模型微调方法。
- 无需特定的注意力机制修改。

**内存建议：**

- 模型大小决定基础内存需求。
- 数据集特征（视频长度、分辨率）会影响内存需求。
- 对于更大的模型，建议使用多 GPU 训练。

#### 输入规格

**视频输入指南：**

- **推荐 FPS**：每秒 4 帧
- **Token 预算**：以 8k token 为中心，在 [6k, 10k] 范围内随机化
- **总像素**：约为 8k × 28 × 28 × 2
- **泛化能力**：模型可处理超出训练边界的输入，但质量可能下降

### 可扩展性考虑

#### 数据库扩展

**向量数据库性能：**

- 支持面向大数据集的水平扩展
- 针对批量数据摄取进行了索引优化
- 资源高效扩展（摄取后可关闭实例）
- 在规模增大时仍可保持快速搜索速度

#### GPU 资源管理

**动态分配：**

- 基于 Ray 的流水线支持多种 GPU 类型。
- 支持动态硬件检测和优化
- 通过遥测高效利用资源
- 支持混合硬件配置。

## 许可与可用性

### Cosmos 模型采用什么许可模式？

Cosmos 世界基础模型依据 **NVIDIA Open Model License Agreement** 提供，该协议允许：

- 商业使用且无需付费
- 无公司规模限制
- 合成数据生成
- 后训练和衍生模型开发
- 模型分发与修改

### 企业选项

#### 开源与企业版

- **模型权重和脚本**：开源且免费
- **基础开发工具**：采用宽松许可证提供
- **企业特性**：通过 NVIDIA AI Enterprise（NVAIE）提供

#### NVIDIA AI Enterprise 功能

- 针对更优推理性能优化的 NIMs
- 更高级的 NeMo 功能与维护
- 专业支持和更新
- 生产部署工具

#### 许可证兼容性

现有的 NVIDIA Omniverse Enterprise（NVOVE）许可证可用于 Cosmos 权益。

### 获取支持

#### 社区资源

- **GitHub Issues**：在相关仓库中报告 bug 并请求功能
- **文档**：每个仓库中的完整指南
- **示例**：参考实现和教程
- **社区论坛**：与其他开发者互动
- **Physical AI 数据集**：可在 Hugging Face 上的 [NVIDIA Physical AI Collection](https://huggingface.co/collections/nvidia/physical-ai) 获取面向自动驾驶、机器人、智能空间和仓储环境的精选数据集

#### 官方渠道

- **NVIDIA Developer Portal**：最新更新和公告
- **build.nvidia.com**：试用模型并访问 NIMs
- **NVIDIA AI Catalog**：增强型文本提示工具和专用模型

#### 企业支持

对于企业部署：

- NVIDIA AI Enterprise 订阅包含专业支持
- 为生产部署提供专门的技术协助
- 提供定期更新和优化
- 提供自定义集成指导

### 法律与合规

#### 许可条款

[NVIDIA Open Model License Agreement](https://developer.download.nvidia.com/licenses/nvidia-open-model-license-agreement-june-2024.pdf) 涵盖：

- 商业使用权
- 分发权限
- 修改许可
- 署名要求

### 常规问题

#### 问：我需要付费才能使用 Cosmos 模型吗？

**答**：不需要，核心模型根据 NVIDIA Open Model License 免费提供。

#### 问：我可以将 Cosmos 用于商业应用吗？

**答**：可以，商业使用不受限制。

#### 问：如果我需要企业级支持怎么办？

**答**：NVIDIA AI Enterprise 提供优化工具和专业支持。

#### 问：是否有任何使用限制？

**答**：使用必须符合 NVIDIA Open Model License Agreement 和适用法律。

#### 问：我可以修改并重新分发这些模型吗？

**答**：可以，许可条款允许修改和再分发。
