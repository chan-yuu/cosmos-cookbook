# 用于多视角仓库检测与跟踪的 Cosmos Transfer 1 Sim2Real

> **Authors:** [Alice Li](https://www.linkedin.com/in/alice-li-17439713b/) • [Thomas Tang](https://www.linkedin.com/in/zhengthomastang/) • [Yuxing Wang](https://www.linkedin.com/in/yuxing-wang-55394620b/) • [Jingyi Jin](https://www.linkedin.com/in/jingyi-jin)
> **Organization:** NVIDIA

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Transfer 1](https://github.com/nvidia-cosmos/cosmos-transfer1) | Inference | Sim to Real data augmentation |

本用例演示了如何将 Cosmos Transfer 1 用于对 Omniverse (OV) 生成的合成数据进行数据增强，以缩小 sim-to-real 域差距，重点面向多视角仓库检测与跟踪场景。

- [安装与系统要求](setup.md)

## 用例说明

NVIDIA Omniverse 是一个强大的合成数据生成（SDG）平台，可对场景生成和仿真进行精确控制。数字仿真虽然能够结合场景控制生成深度图、分割图等精确派生信息，但要为稳健模型训练生成多样化变体，往往计算成本高且耗时。

本用例探讨了如何利用 Cosmos Transfer 1 实现首次 outside-in 多视角世界仿真。将 Omniverse 生成的合成仓库场景转换为逼真的多样化版本，可以在不重新渲染整个场景的情况下缩小 sim-to-real 域差距。

## Outside-In 多视角处理方法

对仓库空间的监控通常涉及多摄像头视角，以提供全面覆盖。由于 Cosmos Transfer 1 原生并不支持多视角处理，我们采用如下方法来确保所有摄像头视角之间的视觉一致性：

1. **多视角 Outside-In 数据生成**：通过 [IsaacSim.Replicator.Agent](https://docs.isaacsim.omniverse.nvidia.com/latest/index.html) 准备多视角合成视频及其对应的多模态真值数据（例如深度图、分割掩码）。
2. **处理**：对于每个视频，都向 Cosmos Transfer 1 模型提供相同的文本提示词和参数设置，以确保不同摄像头视角之间的一致性。我们会仔细选择并分析模态，以增强不同摄像头视角之间目标特征的一致性。同时采用详细、数据驱动的文本提示词，以尽量减少视角间目标特征的偏差。在以下案例中，仅使用 depth 和 edge map（0.5 depth + 0.5 edge）作为 Cosmos Transfer 1 模型的输入控制。

这种方法可确保所有摄像头视角在整个多视角设置中获得一致的环境变换，同时保持空间和时间上的连贯性。

![Data augmentation pipeline](assets/warehouse_mv_pipeline.png)

## 演示概览

本演示展示了 **Cosmos Transfer 1** 如何通过程序化场景随机化（包括图像噪声、光照变化、纹理变化和物体摆放多样性）实现 sim-to-real 域适配。该参数一致框架确保不同摄像头视角之间的变换保持一致，从而提升仓库环境中下游 3D 检测与跟踪性能。

## 数据集与设置

### 仓库输入数据示例

检测和跟踪算法的部分训练数据样本存储在本地 `assets/SURF_Booth_030825/` 目录中。该多摄像头仓库数据条目为仓库场景提供了来自多个摄像头视角的同步渲染 RGB 和深度信息。

数据集位于以下目录：

```
scripts/examples/transfer1/inference-warehouse-mv/assets/SURF_Booth_030825/
```

### 数据结构

数据集提供了一个 6 摄像头仓库设置，同步数据组织如下：

- **`Camera_00/` 到 `Camera_05/`**：各自独立的摄像头目录，每个目录包含以下内容：
  - **`rgb.mp4`**：该摄像头视角的 RGB 视频数据
  - **`depth.mp4`**：该摄像头视角对应的深度视频数据

### 更多 Physical AI Smart Spaces 数据集

如需更多多摄像头仓库数据集，请参阅 Hugging Face 上的 [NVIDIA PhysicalAI-SmartSpaces dataset](https://huggingface.co/datasets/nvidia/PhysicalAI-SmartSpaces)，其中包含超过 250 小时的同步多摄像头视频，以及 2D/3D 标注、深度图和标定数据。

### 仓库 Outside-In 多视角输入

每个摄像头的 RGB 视频都会通过多次 Cosmos Transfer 1 推理顺序处理。下面展示拼接后的多视角视频，以说明组合后的多个视角。

**多视角 RGB 输入：**

<video width="720" controls>
  <source src="assets/combined_grid_rgb.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

**组合后的多视角 Depth 控制：**

<video width="720" controls>
  <source src="assets/combined_grid_depth.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Cosmos Transfer 1 流程组件

### 通过环境变化实现 Sim2Real 转换

我们可以利用 Cosmos Transfer 1 模型将合成计算机图形的外观转换为真实的仓库环境。通过恰当地编写提示词，我们可以在保留结构布局和对象关系的同时，引入不同光照场景等多样化环境条件。

```json
{
  "prompt": "The camera provides a clear view of the warehouse interior, showing rows of shelves stacked with boxes, baskets and other objects. There are workers and robots walking around, moving boxes or operating machinery. The lighting is bright and even, with overhead fluorescent lights illuminating the space. The floor is clean and well-maintained, with clear pathways between the shelves. The atmosphere is busy but organized, with workers and humanoids moving efficiently around the warehouse.",
  "input_video_path": "/your_video_path/rgb_video.mp4",
  "edge": {
    "control_weight": 0.5
  },
  "depth": {
    "control_weight": 0.5,
    "input_control": "/your_video_path/depth_video.mp4"
  }
}
```

**组合后的多视角 Transfer 1 输出：**

<video width="720" controls>
  <source src="assets/combined_grid_output.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

## Cosmos Transfer 1 中的控制参数

通过使用不同环境描述更新文本提示词，Cosmos Transfer 1 除了能缩小 sim-to-reality 差距外，还能提升数据多样性。例如，提供雾天或低照度仓库场景描述，可以生成更具挑战性的场景数据。

### 雾天和昏暗条件

控制与文本提示词示例如下：

```json
{
  "prompt": "The camera provides a photorealistic view of a dimly lit warehouse shrouded in thick fog, where shelves emerge like silhouettes through the mist, holding crates covered in dew. Workers wear diversed layered clothing in muted tones, including utility jackets, waterproof pants, and sturdy boots, some paired with scarves and beanies to counter the chill. Humanoid robots with matte black finishes and faintly glowing outlines navigate through the haze, their movements slow and deliberate. Forklifts, painted in industrial gray with fog lights attached, glide silently across the damp concrete floor. The lighting is eerie and diffused, with beams from overhead fixtures piercing the mist to create dramatic light shafts. The atmosphere is mysterious and quiet, with the muffled sound of machinery barely audible through the thick air.",
  "prompt_title": "The camera provides a photorealistic",
  "input_video_path": "/your_video_path/rgb_video.mp4",
  "edge": {
    "control_weight": 1.0
  }
}
```

![Data augmentation pipeline](assets/foggy_dark_warehouse-min.gif)

### 进一步扩展规模

通过为世界仿真准备多样化的文本提示词，我们可以显著扩大生成场景的种类。

下面的演示展示了 Cosmos Transfer 1 如何使用多个文本提示词增强单一视角，从而体现提示词多样性的影响。

![Data augmentation pipeline](assets/multi_world_simulation-min.gif)

还可以通过将每个摄像头视角视频划分为 10 个片段，并为每个片段分配独特的文本提示词，进一步提升多样性。

![Text Prompt Simulation](assets/viz_grid_text_prompts.jpg)

### 推荐的控制配置

与天气增强方法类似，实验表明，仅控制 _edge 和 depth_ 能够为仓库 sim-to-real 转换带来最佳效果。该配置在允许外观发生真实变化的同时，保持结构一致性。

```json
{
  "prompt": "The camera provides a clear view of the warehouse interior, showing rows of shelves stacked with boxes, baskets and other objects. There are workers and robots walking around, moving boxes or operating machinery. The lighting is bright and even, with overhead fluorescent lights illuminating the space. The floor is clean and well-maintained, with clear pathways between the shelves. The atmosphere is busy but organized, with workers and humanoids moving efficiently around the warehouse.",
  "input_video_path": "/mnt/pvc/gradio/uploads/upload_20250916_152159/rgb.mp4",
  "edge": {
    "control_weight": 0.5
  },
  "depth": {
    "control_weight": 0.5,
    "input_control": "/mnt/pvc/gradio/uploads/upload_20250916_152159/depth.mp4"
  }
}
```

## 增强数据集上的 2D 检测结果

为评估 Cosmos Transfer 1 在数据增强方面的有效性，我们使用 AI City Challenge 数据集中精心挑选的多视角场景进行了实验。从 [AI City v0.1](https://www.aicitychallenge.org/) 数据集中选取了 _11 个不同场景_，代表多样化的仓库和室内环境。

这 11 个基线场景中的每一个都经过前文所述的多视角参数一致增强流程，由 Cosmos Transfer 1 进行处理。该过程在保持结构一致性和多视角连贯性的同时，生成了环境变化、光照变化以及包括粉尘和低能见度在内的环境条件变化。

随后，利用得到的增强数据集（同时包含每个场景的原始版本和经 Cosmos Transfer 增强的版本）训练 RT-DETR 和 EfficientViT-L2 检测器。性能对比显示，计算机视觉（CV）模型的准确性和真实世界泛化能力均有显著提升。

### 检测性能结果

| Dataset Configuration                         | Pretrained Checkpoint | Building K Person AP50 | Building K Nova Carter AP50 | Building K mAP50 |
| --------------------------------------------- | --------------------- | ---------------------- | --------------------------- | ---------------- |
| **Baseline: 1-min IsaacSim AICity v0.1**      | NVImageNetV2 backbone | 0.776                  | 0.478                       | 0.627            |
| **1-min Cosmos AICity v0.1**                  | NVImageNetV2 backbone | 0.827 (+6.17%)         | 0.545 (+12.35%)             | 0.686 (+8.60%)   |
| **1-min IsaacSim + 1-min Cosmos AICity v0.1** | NVImageNetV2 backbone | 0.838 (+7.40%)         | 0.645 (+25.94%)             | 0.742 (+15.50%)  |

## 结论

本用例展示了用户如何将 Cosmos Transfer 1 作为 AI 模型和框架，用于多视角仓库场景中的数据增强，以弥合 sim-to-real 域差距。以下是关键要点：

1. **高性价比的数据增强**：Cosmos Transfer 1 为昂贵的合成数据重新生成提供了高效替代方案，可快速创建环境变化版本。
2. **多视角一致性**：参数一致的方法可在保持空间和时间连贯性的同时，确保所有摄像头视角之间的变换保持一致。
3. **显著的性能提升**：当使用 Cosmos Transfer 1 增强数据集进行训练时，RT-DETR 和 EfficientViT-L2 检测器都表现出明显提升（mAP 提高 8-11%）。
4. **最佳控制配置**：仅使用 edge 和 depth 控制即可获得仓库 sim-to-real 转换的最佳结果。

通过应用该框架，我们可以生成保持多视角一致性的逼真仓库场景，在减少昂贵真实世界数据采集或高成本合成数据重渲染需求的同时，显著提升下游检测与跟踪算法的准确性。

---

## 文档信息

**Publication Date:** October 09, 2025

### 引用

如果你使用了此配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_cosmos_transfer_1_2025,
  title={Cosmos Transfer 1 Sim2Real for Multi-View Warehouse Detection and Tracking},
  author={Li, Alice and Tang, Thomas and Wang, Yuxing and Jin, Jingyi},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer1/inference-warehouse-mv/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**Suggested text citation:**

> Alice Li, Thomas Tang, Yuxing Wang, & Jingyi Jin (2025). Cosmos Transfer 1 Sim2Real for Multi-View Warehouse Detection and Tracking. In _NVIDIA Cosmos Cookbook_. Accessible at <https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer1/inference-warehouse-mv/inference.html>
