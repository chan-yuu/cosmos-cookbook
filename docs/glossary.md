# 术语表

## C

**Chain of Thought (CoT)**
: 一种推理技术，模型在得出最终答案之前，会先逐步生成对其思考过程的解释，从而提升透明度和准确性。

**Checkpoint**
: 模型权重和训练状态在训练过程中某一特定时刻的已保存快照，可用于恢复训练或在不同阶段评估模型。

**Context Parallelism (CP)**
: 一种并行化策略，将序列/上下文维度拆分到多个设备上，以处理更长的序列。

**Cosmos Curator**
: 一个基于 Ray 的 GPU 加速视频整理流水线，用于多模型分析、内容过滤、标注以及推理和训练数据的去重。

**Cosmos Predict**
: 一个用于未来状态预测和 video-to-world 生成的 diffusion transformer 模型，并提供面向机器人和仿真的专用变体。

**Cosmos Reason**
: 一个 7B 视觉语言模型，用于物理世界扎根推理，可处理 embodied AI 应用中的空间/时间理解和 chain-of-thought 任务。

**Cosmos RL**
: 一个分布式训练框架，支持监督微调（SFT）和强化学习方法，并具备弹性策略 rollout 以及 FP8/FP4 精度支持。

**Cosmos Transfer**
: 一个具备 ControlNet 和 MultiControlNet 条件控制（depth、segmentation、LiDAR、HDMap）的多控制视频生成系统，并包含 4K 超分能力。

**ControlNet**
: 一种神经网络架构，可为 diffusion model 增加条件控制，使其能够由深度图、边缘图或分割 mask 等额外输入进行引导。

## D

**Data Parallel (DP)**
: 一种训练并行策略，在多个设备上复制模型，并由每个设备处理不同的数据批次。

**Data Parallelism Shard Size**
: 在分布式训练中，同步梯度的设备数量。

**Deduplication**
: 识别并移除数据集中的重复或近似重复样本，以提升数据质量和训练效率的过程。

**Diffusion Model**
: 一种生成模型，通过从纯噪声开始迭代去噪并逐步细化为连贯输出，学习如何创建数据。

## E

**Embodied AI**
: 通过传感器和执行器与物理世界交互的 AI 系统，例如机器人和自动驾驶汽车。

**Epoch**
: 在模型训练中，对整个训练数据集完成一次完整遍历。

## F

**Fine-Tuning**
: 通过在特定任务数据上继续训练，将预训练模型适配到特定任务或领域的过程。

**FP4/FP8**
: 4 位和 8 位浮点数格式，可在保持可接受模型性能的同时降低内存使用并提升训练速度。

**FPS (Frames Per Second)**
: 每秒处理或生成的视频帧数。

**FSDP (Fully Sharded Data Parallel)**
: 一种内存高效的分布式训练策略，将模型参数、梯度和优化器状态切分到多个设备上。

## G

**Gradient Checkpointing**
: 一种以内存换计算的技术，在反向传播期间重新计算中间激活值，而不是存储它们，从而节省内存。

**Gradient Clipping**
: 一种通过将梯度范数限制在最大值以内来防止梯度爆炸的技术。

## H

**HDMap (High-Definition Map)**
: 自动驾驶中使用的精细车道级地图表示，包含精确道路几何、车道线和交通规则。

## I

**Inference**
: 使用已训练模型在新的、未见过的数据上进行预测或生成输出的过程。

**Interactive Meta-Action**
: 在自动驾驶中，涉及与其他交通参与者交互的驾驶行为，例如让行、跟车或超车。

**ITS (Intelligent Transportation Systems)**
: 旨在为不同交通方式和交通管理提供创新服务的先进应用。

## L

**LoRA (Low-Rank Adaptation)**
: 一种高效微调方法，通过向预训练模型权重添加可训练的低秩矩阵，减少需要训练的参数数量。

**LLM (Large Language Model)**
: 在海量文本数据上训练的神经网络模型，能够理解并生成类似人类的文本。

## M

**Max Pixels**
: 图像或视频帧中可处理的最大像素数量，通常用于控制计算需求。

**Model Checkpoint**
: 参见 **Checkpoint**。

**Multi-Control**
: 同时使用多种控制信号（例如 depth、segmentation 和 HDMap）对生成模型进行条件控制的能力。

**MultiControlNet**
: ControlNet 的扩展版本，结合多个条件控制信号，以更高精度引导视频生成。

## N

**Non-Interactive Meta-Action**
: 在自动驾驶中，不直接涉及其他交通参与者的驾驶行为，例如并道或在空旷路口转弯。

## O

**Optimizer**
: 一种在训练过程中调整模型权重以最小化损失函数的算法。常见优化器包括 Adam、AdamW 和 SGD。

## P

**Parallelism**
: 将计算分布到多个设备上以加速训练或推理的策略。另见 **Data Parallel**、**Tensor Parallel**、**Pipeline Parallel**。

**Physical Plausibility**
: 生成或预测内容遵循真实世界物理规律和约束的程度。

**Pipeline Parallel (PP)**
: 一种训练策略，将模型拆分为分布在多个设备上的顺序阶段，由不同设备处理不同层。

**Post-Training**
: 在初始训练完成后，继续在特定任务或领域上训练或微调预训练模型的过程。

## R

**Reinforcement Learning (RL)**
: 一种机器学习方法，模型根据其动作获得奖励或惩罚，并以长期累计回报最大化为目标进行学习。

**Reward Model**
: 一种用于打分或评估输出的模型，在强化学习中作为反馈信号提供训练依据。

## S

**Scene Understanding**
: 模型解释并理解视觉场景中的内容、上下文及其关系的能力。

**SFT (Supervised Fine-Tuning)**
: 一种训练方法，使用带标签示例和监督学习目标对预训练模型进行微调。

**Sim-to-Real (Sim2Real)**
: 将在仿真环境中学习到的知识或训练出的模型迁移到真实世界应用中的过程。

**Spatial AI**
: 能够理解并推理物理环境中空间关系、位置和交互的 AI 系统。

**System Prompt**
: 提供给语言模型的初始指令或上下文，用于定义其在对话或任务中的角色、行为或约束。

## T

**Tensor Parallel (TP)**
: 一种并行化策略，将单个 tensor（模型层）拆分到多个设备上。

**Traffic Participant**
: 交通环境中的任何实体，包括车辆、行人、自行车骑行者及其他道路使用者。

**Training Configuration**
: 定义模型训练方式的一组超参数和设置，包括学习率、批大小和优化策略。

**Transfer Learning**
: 将从一个任务或领域中学到的知识应用到另一个不同但相关任务或领域，以提升性能的技术。

## U

**Upscaling**
: 提升图像或视频分辨率，同时尽量保持或增强质量的过程。

## V

**Video Augmentation**
: 对视频数据进行修改或增强的技术，例如改变天气、光照或风格，以增加数据集多样性。

**Video-Language Model (VLM)**
: 一种同时处理视频和语言输入的神经网络模型，能够理解视觉内容并生成或响应文本。

**Visual Question Answering (VQA)**
: 一项让模型回答关于图像或视频内容问题的任务。

**VRU (Vulnerable Road User)**
: 未受车辆结构保护的交通参与者，包括行人、自行车骑行者和摩托车骑行者。

## W

**Warmup Steps**
: 训练初期的一个阶段，在此期间学习率从较小值逐步增加到目标学习率，有助于稳定早期训练。

**Weight Decay**
: 一种正则化技术，在损失函数中加入与模型权重大小成比例的惩罚项，以帮助防止过拟合。

**WFM (World Foundation Model)**
: 用于理解和生成物理世界表征的大规模基础模型，是 Cosmos 生态系统的基础。

## Z

**Zero-Shot**
: 模型在未针对某一特定任务接受显式训练示例的情况下，仅依赖预训练知识完成该任务的能力。
