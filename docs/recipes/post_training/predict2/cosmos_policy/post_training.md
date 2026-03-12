# Cosmos Policy：面向视觉运动控制与规划的视频模型微调

> **作者：** [Moo Jin Kim](https://moojink.com) • [Yuzhu Dong](https://www.linkedin.com/in/yuzhudong/) • [Jinwei Gu](https://www.gujinwei.org/)
>
> **机构：** NVIDIA, Stanford University
>
> Cosmos Policy 网站：<https://research.nvidia.com/labs/dir/cosmos-policy/>

## 概述

| **Model** | **Workload** | **Use Case** |
| --- | --- | --- |
| [Cosmos-Predict2-2B-Video2World](https://github.com/nvidia-cosmos/cosmos-predict2) | 后训练 | 基于视觉的机器人操作与基于模型的规划 |
| [Cosmos-Predict2.5-2B-Video2World](https://github.com/nvidia-cosmos/cosmos-predict2.5) | 后训练 | 基于视觉的机器人操作与基于模型的规划 |

![Cosmos Policy](assets/cosmos_policy_figure1.jpeg)

Cosmos Policy 通过在机器人演示数据上进行单阶段后训练，并且**不做任何架构修改**，将 Cosmos-Predict2-2B 和 Cosmos-Predict2.5-2B-Video2World 视频基础模型适配为**最先进的机器人策略**。该方法利用视频模型预训练得到的先验来：

1. **生成机器人动作**
2. **预测未来状态**（机器人本体感知和相机图像）
3. **估计价值**（期望累计回报）

下面展示了 Cosmos Policy 在 RoboCasa 仿真任务中的示例 rollout（上排），以及其未来图像预测（下排）。

<div style="display: flex; gap: 10px; flex-wrap: wrap;">
  <video width="49%" controls autoplay loop muted playsinline src="./assets/robocasa_rollouts/robocasa_rollout_with_future_predictions.mp4"></video>
  <video width="49%" controls autoplay loop muted playsinline src="./assets/robocasa_rollouts/robocasa_rollout_with_future_predictions2.mp4"></video>
</div>

<br>

本配方演示如何使用“latent frame injection”针对机器人操作任务微调 Cosmos-Predict：即将新模态（动作、机器人本体感知状态和未来状态价值）直接编码进视频模型的 latent diffusion 序列中。我们讨论三个示例：LIBERO 仿真基准任务、RoboCasa 仿真基准任务以及真实世界的 ALOHA 机器人任务。

为什么策略需要预测未来状态和价值？这是因为 Cosmos Policy 最初设计为可让用户以两种方式之一部署：

1. 作为直接策略（即直接执行模型生成的动作块，而无需复杂规划）
2. 结合基于模型的规划（即生成多个动作块候选、未来状态和值，并选择价值最高的动作块）。

事实证明，即便将 Cosmos Policy 作为不带任何规划的直接策略部署，也能达到最先进结果；而基于模型的规划则可能进一步提升性能，但代价是更高复杂度和更慢推理速度。因此，为了简化起见，本 cookbook 配方聚焦于前者：训练一个标准的 Cosmos Policy 并将其作为直接策略部署。（关于后者的更多细节可参考 Cosmos Policy 论文。）

## 挑战：将视频模型适配到机器人控制

大型预训练视频模型从数百万视频中学习时间因果、隐式物理和运动模式。然而，先前将视频模型适配到机器人领域的方法通常需要：

* **多个训练阶段**（例如先在机器人数据上进行视频微调，再训练动作模块）
* **新的架构组件**（例如独立的动作 diffuser 或逆动力学模型）
* **无法利用预训练视频先验的自定义模型设计**（例如 Unified Video Action model 和 Unified World Model）

相比之下，Cosmos Policy 采用了更简单的方法：

* **单阶段训练**，直接在机器人演示数据上训练
* **无需对基础视频模型做架构改动**
* **在同一统一架构中联合建模策略、世界模型和值函数**

## 方法：Latent Frame Injection

### 核心概念

Cosmos Policy 将新的输入/输出模态直接编码为视频模型 latent diffusion 序列中的 latent frames。给定视频模型的 $(1 + T') \times H' \times W' \times 16$ latent 序列，我们交织插入：

* **机器人本体感知**（机器人关节角或末端执行器位姿）
* **动作块**（跨越多个时间步的动作序列）
* **未来状态价值**（期望累计回报）
* **多相机视角图像**（例如第三人称视角、腕部相机）

该 latent 序列遵循 $(s, a, s', V(s'))$ 结构。下面是 RoboCasa 仿真任务的 latent 序列示例。绿色的“conditioning subsequence”帧用于条件输入模型生成；红色的“target subsequence”帧是模型通过 reverse diffusion 去噪/生成时训练的目标。

<video width="100%" controls autoplay loop muted playsinline src="assets/cosmos_policy_latent_diffusion_sequence.mp4"></video>

| Position | Content | Description |
| --- | --- | --- |
| 0 | （上图未显示）Blank/Null | 占位 latent frame（用于兼容视频模型的 $(1 + T')$ 时间压缩） |
| 1 | Current proprio | 机器人本体感知状态 |
| 2-4 | Current images | 1 个腕部相机 + 2 个第三人称相机 |
| 5 | Action chunk | 预测的 $K \times d_{act}$ 动作 |
| 6 | Future proprio | 预测的机器人本体感知状态（“future state”的一部分） |
| 7-9 | Future images | 预测的相机视图（“future state”的一部分） |
| 10 | Value | 从预测未来状态出发的期望 rewards-to-go |

对于 LIBERO 和 ALOHA，latent diffusion 序列略有不同。例如，在 LIBERO 中，只有两个相机图像（一个第三人称相机和一个腕部相机），因此总 latent frame 少两个。对于 ALOHA，则有三个相机图像，但分别来自一个第三人称相机和两个腕部相机。

### 为 Latent Injection 预处理非图像模态

对于机器人本体感知、动作和值，在训练 Cosmos Policy 之前，我们进行如下预处理：

1. 将每种模态**归一化**到 $[-1, +1]$
2. 将其**展平**为向量（例如动作块：$K \times d_{act}$）
3. 将向量**复制**足够次数，以填满对应的 latent volume $(H' \times W' \times C')$
4. 通过覆盖占位 latent frames 来进行**注入**

在推理时，我们从 latent volumes 中提取预测结果；由于其中存在重复项，我们只需对所有重复项取平均，再反归一化回原始模态的尺度。

## 训练方法

### 联合训练目标

Cosmos Policy 在单一统一模型中通过平衡的 batch 划分方案联合学习策略、世界模型和值函数：

| Objective | Batch % | Learns |
| --- | --- | --- |
| Policy | 50% | $p(a, s', V(s') \mid s)$ |
| World Model | 25% | $p(s', V(s') \mid s, a)$ |
| Value Function | 25% | $p(V(s') \mid s, a, s')$ |

conditioning 方案决定哪些 latent frames 是 clean（conditioning），哪些是 noised（target）：

