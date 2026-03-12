# 面向机器人导航任务的 Cosmos Transfer Sim2Real

> **Authors:** [Aigul Dzhumamuratova](https://www.linkedin.com/in/aigul-dzhumamuratova-78232b234/) • [Hesam Rabeti](https://www.linkedin.com/in/hesamrabeti/) • [Yan Chang](https://www.linkedin.com/in/yanchang1/) • [Jingyi Jin](https://www.linkedin.com/in/jingyi-jin)
> **Organization:** NVIDIA

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Transfer 1](https://github.com/nvidia-cosmos/cosmos-transfer1) | Inference | Sim to Real data augmentation |

本教程演示如何将 Cosmos Transfer 应用于机器人导航任务，以提升 Sim2Real 性能。

- [安装与系统要求](../inference-warehouse-mv/setup.md)

## 用例说明

在本次评估中，我们使用了多个 NVIDIA 工具——[X-Mobility](https://nvlabs.github.io/X-MOBILITY/) 导航模型和 [Mobility Gen](https://github.com/NVlabs/MobilityGen) 数据生成工作流。

1. **X-Mobility** 是一个端到端导航模型，能够在多种环境之间实现泛化。它使用 on-policy 和 off-policy 数据来学习导航策略。原始 X-Mobility 数据集完全通过 Mobility Gen 在模拟仓库和室内环境中创建。
2. **Mobility Gen** 是一个基于 NVIDIA Isaac Sim 构建的工作流，用于为移动机器人（包括轮式、四足和人形平台）生成高保真的合成运动与感知数据。它可产生详细的真值标注，例如占据图、机器人位姿与速度、RGB 与深度图像以及语义分割掩码，从而支持可扩展的数据集创建，用于导航与控制策略训练。

我们使用原始 X-Mobility 数据集中的 RGB 和分割数据作为 Cosmos Transfer 的输入。Cosmos Transfer 通过改变光照条件、障碍物外观和材质属性来增强数据集，同时不改变原始几何结构或机器人运动。这使得轨迹和语义分割等真值信息得以保留，同时生成更逼真、更多样化的数据集，用于训练稳健的导航模型。

![Data augmentation pipeline overview](assets/pipeline1.png)

## 演示概览

本演示展示了 **Cosmos Transfer 1** 如何实现 Sim2Real 域适配。它提供了一个分步工作流，用于将 Mobility Gen 数据集转换为更具照片真实感和多样性的版本，在增强数据上训练模型，并最终在真实世界条件下评估其性能。

## 数据集与设置

### **X-Mobility 数据集**

X-Mobility Dataset 可从 Hugging Face 下载：[https://huggingface.co/datasets/nvidia/X-Mobility](https://huggingface.co/datasets/nvidia/X-Mobility)。这个具备照片真实感的合成数据集提供了两类动作策略输入：随机动作和基于 Nav2 的教师策略：

- **Random Action Dataset** (`x_mobility_isaac_sim_random_160k.zip`)：160K 帧，用于不带动作网络的世界模型预训练
- **Teacher Policy Dataset** (`x_mobility_isaac_sim_nav2_100k.zip`)：100K 帧，用于联合训练世界模型和动作策略

每一帧都包含关键字段：image、speed、semantic label、route、path 和 action command。其中 image 字段包含前视相机的 RGB 输入，而 semantic label 字段根据预定义语义类别标识每个像素：[Navigable, Forklift, Cone, Sign, Pallet, Fence, and Background]

> **Note:** 本实验中使用的所有脚本均位于 `$COOKBOOK_ROOT/scripts/examples/transfer1/inference-x-mobility/`，供读者参考。

## Cosmos Transfer 数据增强流程

### **将 X-Mobility 数据集转换为 Cosmos Transfer 输入视频**

我们使用 X-Mobility 数据集中的 RGB 和分割数据作为 Cosmos Transfer 的输入。运行以下命令，为 Cosmos Transfer 准备视频输入：

```shell
uv run scripts/examples/transfer1/inference-x-mobility/xmob_dataset_to_videos.py data/x_mobility_isaac_sim_nav2_100k data/x_mobility_isaac_sim_nav2_100k_input_videos

uv run scripts/examples/transfer1/inference-x-mobility/xmob_dataset_to_videos.py data/x_mobility_isaac_sim_random_160k data/x_mobility_isaac_sim_random_160k_input_videos
```

该过程会创建两个视频目录，并镜像原始数据集的目录结构。每个样本都包含一个 RGB `<video_file>.mp4` 以及一个对应的分割视频 `<video_file>_segmentation.mp4`。

### **Cosmos Transfer 多模态控制**

接下来，Cosmos Transfer 1 使用以下命令，根据输入视频生成具备照片真实感的视频：

```shell
export CUDA_VISIBLE_DEVICES="${CUDA_VISIBLE_DEVICES:=0}"
export CHECKPOINT_DIR="${CHECKPOINT_DIR:=./checkpoints}"
export NUM_GPU="${NUM_GPU:=1}"

PYTHONPATH=$(pwd) torchrun --nproc_per_node=$NUM_GPU --nnodes=1 --node_rank=0 cosmos_transfer1/diffusion/inference/transfer.py --checkpoint_dir $CHECKPOINT_DIR  --video_save_folder outputs/example --controlnet_specs assets/inference_cosmos_transfer1_custom.json --offload_text_encoder_model --offload_guardrail_models --num_gpus $NUM_GPU
```

示例控制与文本提示词 `inference_cosmos_transfer1_custom.json`：

```json
{
    "prompt": "A realistic warehouse environment with consistent lighting, perspective, and camera motion. Preserve the original structure, object positions, and layout from the input video. Ensure the output exactly matches the segmentation video frame-by-frame in timing and content. Camera movement must follow the original path precisely.",
    "input_video_path" : "data/output_0360.mp4",
    "edge": {
        "control_weight": 0.3,
    },
    "seg": {
        "input_control": "data/output_0360_segmentation.mp4",
        "control_weight": 1.0
    }
}
```

分割控制权重设置为 1.0，以保留原始几何结构和运动，从而能够复用原始 X-Mobility 数据集中的真值标签。下图展示了该过程：上排为输入 RGB 视频及其作为控制的对应分割掩码，下排为 Cosmos Transfer 1 使用上述 JSON 规范生成的具备照片真实感的输出。

![Input video with the segmentation control and its output](assets/output_xmob.gif)

## 使用 Cosmos 增强数据训练 X-Mobility

### **将 Cosmos Transfer 输出转换回 X-Mobility 格式**

在使用 Cosmos Transfer 生成具备照片真实感的视频之后，我们需要将其转换回 X-Mobility 数据集格式，以便用于训练。`cosmos_dataset_to_xmob.py` 脚本通过以下方式重建原始 X-Mobility 数据结构：

1. 从 Cosmos 生成的视频中提取单独帧
2. 从原始数据集中复制真值标注（语义标签、routes、paths、actions）
3. 将数据组织为包含所有必需字段的 X-Mobility 格式

运行以下命令，对两个数据集执行转换：

```shell
# Convert the random action dataset (for world model pre-training)
python cosmos_dataset_to_xmob.py \
  --input_dir_cosmos data/x_mobility_isaac_sim_random_160k_cosmos_output \
  --input_dir_xmob data/x_mobility_isaac_sim_random_160k \
  --output_dir data/x_mobility_isaac_sim_random_160k_cosmos_to_xmob \
  --mode simple

# Convert the teacher policy dataset (for joint world model and action policy training)
python cosmos_dataset_to_xmob.py \
  --input_dir_cosmos data/x_mobility_isaac_sim_nav2_100k_cosmos_output \
  --input_dir_xmob data/x_mobility_isaac_sim_nav2_100k \
  --output_dir data/x_mobility_isaac_sim_nav2_100k_cosmos_to_xmob \
  --mode segmented
```

**参数说明：**

- `--input_dir_cosmos`：包含 Cosmos Transfer 输出视频的目录
- `--input_dir_xmob`：包含原始 X-Mobility 数据集的目录（真值标注来源）
- `--output_dir`：输出目录，用于存放转换后的 X-Mobility 格式数据集
- `--mode`：转换模式（`simple` 表示基础转换，`segmented` 表示处理视频片段，例如 Teacher Policy Dataset `x_mobility_isaac_sim_nav2_100k_cosmos_to_xmob`）

由于生成时将分割控制权重设置为 1.0，几何结构和运动会被精确保留，因此可以直接复用原始数据集中的所有真值标签。

### **训练配置**

我们按 **相同比例（1:1）** 混合原始数据和 Cosmos 增强数据，以创建混合数据集。模型按照 X-Mobility 的训练方法，采用两阶段方式进行训练：

| **Training Stage** | **Dataset** | **Epochs** | **Batch Size** | **Hardware** | **Additional Notes** |
|-------------------|-------------|------------|----------------|--------------|---------------------|
| **Stage 1: World Model Pre-training** | Random action dataset (160K frames) | 100 | 32 | 8× H100 GPUs | Pre-train world model without action network |
| **Stage 2: Action Policy Training** | Teacher policy dataset (100K frames) | 100 | 32 | 8× H100 GPUs | Joint training with world model; RGB diffuser disabled for speed |

**关键数据集构成：**

- **混合数据集** = 50% 原始 X-Mobility + 50% Cosmos 增强数据
- **总训练数据** = 520K 帧（260K 原始数据 + 260K 增强数据）

下图展示了完整训练流程：使用混合数据集（按相同比例混合原始数据和 Cosmos 增强数据）完成世界模型预训练和动作策略训练两个阶段，从而提升成功率。

![Training pipeline diagram](assets/training.png)

## 在 Isaac Sim 和真实世界中测试

为了评估 Cosmos Transfer 在数据增强中的有效性，我们在 Isaac Sim 仿真环境以及真实世界的 NVIDIA Carter 机器人上测试了训练好的模型。
在 Isaac Sim 中测试表明，原始模型与增强模型的性能相当。而在真实世界闭环评估中，使用混合数据集（Mobility Gen + Cosmos）训练的 X-Mobility 策略明显优于仅使用 Mobility Gen 数据训练的基线模型。

改进后的策略成功应对了以下具有挑战性的场景：

- 绕过透明障碍物导航
- 避开低对比度障碍物（例如灰色地面上的灰色杆）
- 更贴近障碍物行进，从而减少到目标的总距离
- 在昏暗环境中导航
- 穿越狭窄空间

**透明箱体导航：** 该动画序列展示了并排对比：基线模型（仅基于 X-Mobility 数据训练）无法检测并避开透明箱体障碍物，最终发生碰撞；相比之下，使用 Cosmos 增强数据训练的模型能够成功识别透明障碍物并绕行，体现出其对透明物体这一困难目标更强的感知能力。

![Improved transparent box avoidance](assets/box.gif)

**杆状障碍避让：** 该序列展示了面对高而细的杆状障碍物时的类似改进。基线模型难以检测狭窄的杆并与之碰撞，而 Cosmos 增强模型能够成功感知障碍物并平稳绕行，显示出其对低对比度和几何上更具挑战性目标的检测能力提升。

![Improved thin pole avoidance](assets/pole.gif)

我们为所有这些真实世界测试收集了以下关键指标：

1. 任务成功率 - 机器人到达目标的试验占比。
2. 加权行程时间，通过用行程时间除以成功率来衡量导航效率。
3. 平均绝对角加速度（Average AAA），作为运动平滑性的指标。

### 真实世界导航结果

| Dataset Configuration | Success Rate (%) | Weighted Trip Time (s) | Average AAA (rad/s²) |
|----------------------|------------------|------------------------|----------------------|
| **X-Mobility (Baseline)** | 54 | 58.1 | 0.453 |
| **X-Mobility + Cosmos** | 91 | 25.5 | 0.606 |
| **Improvement** | +68.5% | -56.1% | +33.8% |

## 结论

本演示表明，Cosmos Transfer 能显著提升 Sim2Real 导航性能。采用混合训练方法（按相同比例组合原始数据和 Cosmos 增强数据）取得了以下效果：

- **任务成功率提升 +68.5%**（54% → 91%）
- **行程时间降低 56%**（58.1s → 25.5s）
- **针对透明、低对比度和低照度障碍物的鲁棒性增强**

这些结果凸显了 Cosmos Transfer 通过具备照片真实感的数据增强来提升机器人导航系统鲁棒性和真实世界适应性的强大价值。

---

## 文档信息

**Publication Date:** October 27, 2025

### 引用

如果你使用了此配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_cosmos_transfer_sim2real_2025,
  title={Cosmos Transfer Sim2Real for Robotics Navigation Tasks},
  author={Dzhumamuratova, Aigul and Rabeti, Hesam and Chang, Yan and Jin, Jingyi},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer1/inference-x-mobility/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**Suggested text citation:**

> Aigul Dzhumamuratova, Hesam Rabeti, Yan Chang, & Jingyi Jin (2025). Cosmos Transfer Sim2Real for Robotics Navigation Tasks. In *NVIDIA Cosmos Cookbook*. Accessible at <https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer1/inference-x-mobility/inference.html>
