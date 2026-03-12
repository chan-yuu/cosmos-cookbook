# 使用 Cosmos Transfer 2.5 的多控制配方

> **作者：** [Aiden Chang](https://www.linkedin.com/in/aiden-chang/) • [Akul Santhosh](https://www.linkedin.com/in/akulsanthosh/)
> **机构：** NVIDIA

## 概述

你可以使用这个 [Brev 实例](https://brev.nvidia.com/launchable/deploy?launchableID=env-36ZkptMd4kDDzFGMYN9lz9Ixnel) 体验这些配方，并将其自定义到你自己的视频上。它已完成全部预配置，并会自动提供所需算力，让你可以立即开始。

[![Brev 实例](./assets/nv-lb-dark.svg)](https://brev.nvidia.com/launchable/deploy?launchableID=env-36ZkptMd4kDDzFGMYN9lz9Ixnel)

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| [Cosmos Transfer 2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5) | 推理 | 用于背景替换、光照调整、物体变换以及颜色/纹理修改的多控制视频编辑 |

继续之前，请务必阅读以下关于控制模态的核心概念：
[控制模态概览](../../../../core_concepts/control_modalities/overview.md)。在继续之前，理解每一种控制模态都很重要。

Cosmos Transfer 2.5 支持通过多种控制模态（Edge、Segmentation、Vis）以及掩码能力进行精确视频编辑。本 cookbook 提供了四个适用于常见视频编辑任务的关键配方，每个都针对可预测且高质量的结果进行了优化。

## 控制模态参考

在深入各个配方之前，先了解我们将使用的基本控制模态。继续之前，请务必阅读以下关于控制模态的核心概念：
[控制模态概览](../../../../core_concepts/control_modalities/overview.md)：

| **控制类型** | **说明** | **示例** |
|-----------------|-----------------|-------------|
| **原始视频** | 源视频输入 | <video src="assets/wave.mp4" controls width="300"></video> |
| [**Edge**](#generating-a-more-detailed-edge-control-modality) | Canny 边缘检测输出 | <video src="assets/edge.mp4" controls width="300"></video> |
| [**Filtered Edge**](#generating-a-filtered-edge) | 应用掩码后的边缘（仅保留所需边缘） | <video src="assets/filtered.mp4" controls width="300"></video> |
| **Segmentation** | 语义分割图 | <video src="assets/seg.mp4" controls width="300"></video> |
| **Vis** | 来自原始视频的视觉特征 | <video src="assets/vis.mp4" controls width="300"></video> |
| **Mask** | 二值掩码（白色 = 允许更改） | <video src="assets/mask.mp4" controls width="300"></video> |
| [**Inverted Mask**](#generating-an-inverted-mask) | 掩码反相（白色 = 背景） | <video src="assets/mask_inverted.mp4" controls width="300"></video> |

注意：Edge 和 Vis 可以在运行时自动计算。

<!-- To compute all other modalities, check out the [Control Modalities Summary](../../../../core_concepts/control_modalities/overview.md) and [some starter code if necessary](https://github.com/aiden200/cosmos_transfer_2.5_data_preprocessing). -->

## **快速配方：常见用例**

将此表作为你项目的起点。

| 任务 | 建议的控制与设置 | 原始视频 | 增强视频 |
| :---- | :---- | :---- | :---- |
| **更换服装或纹理** | Edge: 1, Guidance: 3 | <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/color.mp4" controls width="300"></video> |
| **改变光照** | Guidance: 3, Edge: 1 + Vis: 0.2 | <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/lighting.mp4" controls width="300"></video> |
| **更换背景，保留主体** | Guidance: 3, Edge Filtered: 1.0 \+ Seg (Mask Inverted): 0.4 + Vis:（中等权重，例如 0.6） | <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/ocean.mp4" controls width="300"></video> |
| **修改物体，但保持真实感** | Guidance: 3, Edge: 0.2 \+ Seg (Mask): 1.0 + Vis:（中等权重，例如 0.5） | <video src="assets/humanoid.mp4" controls width="300"></video> | <video src="assets/object_change.mp4" controls width="300"></video> |

---

## 配方 1：背景更换

<!-- TODO Simulation augmentation -->

### 概述

在保留前景主体及其运动的同时替换视频背景。这个配方非常适合在无需重拍的情况下，将主体放置到新的环境中。

<img src="./assets/background_change_recipe.png" controls width="1300"></img>

### 示例结果

| 原始 | 已更换背景 |
|----------|----------|
| <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/street_background.mp4" controls width="300"></video>  |

### 流水线配置

```json
{
  "name": "change_background",
  "prompt_path": "prompt.txt",
  "video_path": "original.mp4",
  "guidance": 3,
  "edge": {
    "control_weight": 1.0,
    "control_path": "filtered_edge.mp4"
  },
  "seg": {
    "control_weight": 0.4,
    "control_path": "segmentation.mp4",
    "mask_path": "mask_inverted.mp4"
  },
  "vis": {
    "control_weight": 0.6
  }
}
```

### 分步流程

#### 1. 生成 Filtered Edge

仅提取你想保留对象的边缘（例如 human、table）。在这个示例中，我们只想保留挥手的人，并修改场景中的其他所有内容。因此，我们会生成一个 *filtered edge map* 来隔离人的边缘。有关生成 filtered edges 的更多细节，请参见[此部分](#generating-a-filtered-edge)。

仅为需要保留的对象提取边缘（human、table 等）。

<img src="./assets/filtered_edge_recipe.png" controls width="1300"></img>

#### 2. 创建 Inverted Mask

请参考[此部分](#generating-an-inverted-mask)来对掩码进行反相。

<video src="assets/mask_inverted.mp4" controls width="300"></video>

#### 3. 配置控制

1. **Edge (1.0)**：完整保留主体结构
2. **Seg (0.4) + Inverted Mask**：允许生成逼真的背景
3. **Vis (0.6)**：保持光照一致性和鱼眼畸变

#### 4. 结果

- 原始背景被完全替换
- 主体运动和结构得到保留
- 保留了真实的光照与透视
- 保留了鱼眼镜头畸变

#### 5. 调试

- 如果存在一些原始视频的伪影，请调低 `vis`
- 调高 `seg` 以生成更复杂的背景
- 缩放/比例不正确：Depth control 可能可以解决这个问题，但这似乎属于边缘情况

下面是使用 “ocean” 作为背景生成的另一个结果。由于背景伪影，我不得不调低 `vis`。

| 原始视频 | 高 Vis | 低 Vis |
|----------|----------|----------|
| <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/ocean_high_vis.mp4" controls width="300"></video> | <video src="assets/ocean.mp4" controls width="300"></video>  |

作为参考，下面是 ocean prompt：

```txt
A realistic, static full-body shot of a young man standing outdoors near the coast. He has short dark hair and is dressed casually in a dark grey t-shirt, loose black pants, and white sneakers, with an ID badge clipped to his waistband. He faces the camera directly and waves his right hand continuously in a friendly greeting. The surrounding environment is bright and open. In the background, a vast ocean stretches out toward the horizon, with gentle waves, shimmering reflections, and a clear blue sky above. A coastal walkway with railings and scattered pedestrians lines the foreground, replacing the busy city street elements. Soft natural lighting from the sun enhances the calm, breezy seaside atmosphere.
```

## 配方 2：光照变化

<!-- TODO Simulation augmentation -->

### 概述

在保持物体结构和构图不变的情况下，修改场景光照条件（例如白天转夜晚、室内转室外光照）。

<img src="./assets/lighting_change_recipe.png" controls width="600"></img>

### 示例结果

| 原始 | 已改变光照 |
|----------|----------|
| <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/lighting.mp4" controls width="300"></video>  |

### 流水线配置

```json
{
  "name": "change_lighting",
  "prompt_path": "lighting_prompt.txt",
  "video_path": "original.mp4",
  "guidance": 3,
  "edge": {
    "control_weight": 1.0
  },
  "vis": {
    "control_weight": 0.2
  }
}
```

### 对比：有 vis 与无 vis

| 原始视频 | 仅 Edge | Edge + Vis |
|----------|----------|----------|
| <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/lighting_no_vis.mp4" controls width="300"></video> | <video src="assets/lighting.mp4" controls width="300"></video> |

### 配置控制

1. **Edge (1.0)：** 完整保留结构
2. **Vis (0.2)：** 适量加入，以获得更真实的光照物理效果
3. **无 Segmentation：** 全局光照变化不需要它

以下是所使用的 prompt：

```
A realistic, static full-body shot of a young man standing in the center of a spacious, modern atrium. He has short dark hair and is dressed casually in a dark grey t-shirt, loose black pants, and white sneakers, with an ID badge clipped to his waistband. He faces the camera directly and waves his right hand continuously in a friendly greeting. The surrounding space is bright and open, featuring a high industrial-style ceiling with exposed white beams and large, angular black structural supports. The floor is polished light grey concrete, subtly reflecting the warm, soft afternoon sunlight that pours in from large windows above. The overall lighting has a gentle golden tint, with natural shadows stretching slightly to the side in the way they do during late afternoon. In the background, a mezzanine level with glass railings is visible, along with several modern wooden benches and tables scattered throughout the area.
```

## 配方 3：颜色/纹理变化

<!-- TODO Simulation augmentation -->

### 概述

在不改变结构的情况下修改特定物体的颜色或纹理。这是最简单的配方，非常适合产品变体。

<img src="./assets/color_change_recipe.png" controls width="400"></img>

### 示例结果

| 原始 | 已改变颜色/纹理 |
|----------|-------------------|
| <video src="assets/wave.mp4" controls width="300"></video> | <video src="assets/color.mp4" controls width="300"></video>  |

### 流水线配置

```json
{
  "name": "color_change",
  "prompt_path": "color_prompt.txt",
  "video_path": "original.mp4",
  "guidance": 3,
  "edge": {
    "control_weight": 1.0
  }
}

```

### 为什么不使用 Vis Control？

1. Vis 会保留原始颜色/纹理
2. 纯 Edge control 允许在保持结构的同时改变颜色
3. 代价：可能会在其他区域看到轻微色偏

### 配置控制

1. **Edge (1.0)：** 完整保留结构。
2. 在 prompt 中明确说明需要更改的内容。

以下是所使用的 prompt：

```
A realistic, static full-body shot of a young man standing in the center of a spacious, modern atrium. He has short dark hair and is dressed casually in a red t-shirt, loose black pants, and white sneakers, with an ID badge clipped to his waistband. He faces the camera directly and waves his right hand continuously in a friendly greeting. The surrounding space is bright and open, featuring a high industrial-style ceiling with exposed white beams and large, angular black structural supports. The floor is polished light grey concrete, reflecting the artificial overhead lighting. In the background, a mezzanine level with glass railings is visible, along with several modern wooden benches and tables scattered throughout the area.
```

**白色生成问题：** 我们观察到在生成白色时会出现一些问题，但这同样属于边缘情况。

## 配方 4：物体变化

<!-- TODO Simulation augmentation -->

### 概述

在保持真实交互和物理合理性的同时变换特定物体。非常适合产品变体或创意编辑。

<img src="./assets/object_change_recipe.png" controls width="1300"></img>

### 示例结果

| 原始 | 已改变物体 |
|----------|-------------------|
| <video src="assets/humanoid.mp4" controls width="300"></video> | <video src="assets/object_change.mp4" controls width="300"></video>  |

### 流水线配置

```json
{
  "name": "object_change",
  "prompt_path": "object_prompt.txt",
  "video_path": "original.mp4",
  "guidance": 3,
  "edge": {
    "control_weight": 0.2
  },
  "seg": {
    "control_weight": 1.0,
    "control_path": "segmentation.mp4",
    "mask_path": "object_mask.mp4"
  },
  "vis": {
    "control_weight": 0.5
  }
}

```

### 分步流程

#### 1. 生成标准 Edge Map

与背景替换不同，我们使用*完整 edge map*，但给予较低权重（0.2）。这允许模型偏离原始物体结构，同时保持场景连贯性。

<video src="assets/object_edge_full.mp4" controls width="300"></video>

**为什么使用低 edge weight？** 如果你要把一袋薯片变成一个西瓜，形状必须发生巨大变化。高 edge weight 会迫使模型保留袋子的形状，从而产生不真实的结果。

#### 2. 创建对象掩码

生成一个包含目标物体**以及**任何交互元素（例如 robotic gripper）的掩码。白色像素表示模型可以修改的区域。该掩码应包含：

1. 主要物体（例如 vegatable）- 白色像素
2. 交互元素（例如 robot hand）- 白色像素
3. 其他所有内容 - 黑色像素

<video src="assets/object_mask.mp4" controls width="300"></video>

#### 3. 生成 Segmentation

使用完整的 segmentation map 来提供对场景的语义理解：

<video src="assets/object_seg.mp4" controls width="300"></video>

#### 4. 配置控制

1. **Edge (0.2)：** 低权重允许形状变化
2. **Seg (1.0) + Object Mask：** 对掩码区域施加强分割控制
3. **Vis (0.5)：** 中等权重，在真实感与变换能力之间取得平衡

#### 5. 调试

- 抓取看起来不真实 → 扩大接触点周围的掩码
- 物体形状过于受限 → 将 edge weight 降到 0.1
- 背景发生变化 → 将 vis weight 提高到 0.7

作为参考，下面是物体变化的 prompt：

```txt
A first-person point-of-view video from a dual-arm robotic system operating in a research lab. The camera is positioned between the robot's two black, multi-fingered hands, which are visible in the lower corners. The left robotic hand is stationary and is already holding a green bell pepper. On the wooden table in front of the robot, there is an assortment of artificial fruits and vegetables, including two yellow bananas, a bunch of green okra, purple grapes, a green avocado-like object, a grey metal rod, and an orange pear-shaped fruit. The primary action of the video follows the right robotic hand. It starts by moving towards the grey metal rod on the table. The hand's fingers then actuate, closing around the metal rod to grasp it securely. The hand lifts the metal rod off the table and begins moving it towards the right, presumably to place it in a multi-tiered black wire basket that is visible on the right side of the frame and already contains a red apple. The background shows a large, open-plan workshop or lab setting with a gray floor marked by hazard tape, a person sitting at a desk (possibly an operator), other computer workstations, and various pieces of equipment.
```

---

## 额外模态生成

### 生成 Filtered Edge

在实践中，我们发现先使用掩码对 edge 模态进行预过滤，比直接向模型传入 `mask_path` 更可靠。将其作为预处理步骤执行，可以得到更干净、更可控的 edge 信号。

要生成 filtered edge 视频，我们只需将 mask 视频与原始 edge 视频组合起来。过程如下：

<img src="./assets/filtered_edge_recipe.png" controls width="1300"></img>

<!-- Some example code of how to create this can be found [here](https://github.com/aiden200/cosmos_transfer_2.5_data_preprocessing/blob/main/utils/filter_out_edges.py). -->

### 生成 Inverted Mask

为了创建 inverted mask（即白色变黑色，黑色变白色），我们只需对原始 mask 视频应用颜色反转。可以用 ffmpeg 快速完成：

```bash
ffmpeg -y -i mask.mp4 \
  -vf "negate" \
  -c:v libx264 -pix_fmt yuv420p mask_inverted.mp4
```

### 生成更详细的 edge control modality

当物体与背景的轮廓过于相似时，边缘可能无法被可靠检测。在这种情况下，在运行 Canny edge detection 之前提高视频的亮度和对比度，有助于生成更详细、更稳定的 edge map。

<!-- An example implementation of this preprocessing step can be found [here](https://github.com/aiden200/cosmos_transfer_2.5_data_preprocessing/blob/main/control_net_generation/get_object_edges.py). -->

## 从 Omniverse 生成逼真数据

“Sim-to-Real” 是机器人中的一个重要工作流。NVIDIA Omniverse 能够生成合成数据，而我们可以使用 CT 2.5 添加真实世界的域随机化（新的光照、纹理、背景），并生成照片级真实场景。[这里有一些文档](https://docs.isaacsim.omniverse.nvidia.com/5.1.0/introduction/quickstart_index.html)帮助你开始使用基于 NVIDIA Omniverse 构建的 IsaacSim。

工作流如下：

1. 在 Omniverse 中生成：创建一个基础场景（例如机器人抓取一个方块）并导出视频。
2. 提取 Ground Truth：同时从 Omniverse 导出完美的 ground-truth 模态（Depth、Segmentation、Edge）。
3. 使用 CT 2.5 进行增强：利用这些完美的合成控制，并配合新的 prompt（例如 “in a dimly lit warehouse”）运行 CT 2.5。
4. 使用 Cosmos Writer 打包：将新的增强视频与原始 ground-truth controls 一起保存。这会教会下游模型将这些 ground-truth controls 与新的真实风格关联起来。

| 任务 | 建议的控制与设置|
|--|--|
|照片级真实生成|Edge: 1.0 + Seg (Mask Prompt): 0.9 + Depth: 0.9, Guidance: 7|
|创建逼真的背景|Filtered Edge: 1.0 + Seg (Mask Inverted): 0.6 + vis: 0.2, Guidance: 3|

### Omniverse 控制模态

在前一个练习中，我们学习了如何从 Omniverse 提取控制模态。我们从以下控制模态开始：

| 原始视频 | Edge | Seg | Mask |
|----------|----------|----------|----------|
| <video src="./assets/ov_rgb_video.mp4" controls width="300"></video> | <video src="./assets/generated_edges.mp4" controls width="300"></video> | <video src="./assets/segmentation_video.mp4" controls width="300"></video> | <video src="./assets/bw_seg_video.mp4" controls width="300"></video> |

### 照片级真实生成

让我们把模拟环境改造成真实外观

```json
{
  "name": "omniverse_photorealistic",
  "prompt_path": "prompt.txt",
  "video_path": "original.mp4",
  "guidance": 7,
  "edge": {
    "control_weight": 1.0,
    "control_path": "edges.mp4"
  },
  "seg": {
    "control_weight": 0.9,
    "control_path": "segmentation.mp4",
    "mask_prompt": "battered orange safety cone"​
  },
  "depth": {
    "control_weight": 0.9,
    "control_path": "segmentation.mp4",
    "mask_path": "mask_inverted.mp4"
  }
}
```

#### 配方

<img src="./assets/omniverse_photorealistic_recipe.png" width=1300/>

作为参考，下面是该 prompt：

```txt
A humanoid robot walks over to a box and grabs it with its grippers. The warehouse is lit by warm sunlight filtering through windows behind the camera. The floor is beautiful rich walnut flooring. The robot is made of brushed aluminium. Some green boxes are stacked on the left side of the warehouse. Everything in the warehouse is well organized and tidy.
```

### 照片级真实生成结果

| 原始 | 已更换背景 |
|----------|----------|
| <video src="./assets/ov_rgb_video.mp4" controls width="300"></video> | <video src="./assets/omniverse_photorealistic.mp4" controls width="300"></video>  |

### 创建逼真的背景

现在让我们使用以下配置来修改背景

```json
{
  "name": "omniverse_change_background",
  "prompt_path": "prompt.txt",
  "video_path": "original.mp4",
  "guidance": 3,
  "edge": {
    "control_weight": 1.0,
    "control_path": "filtered_edge.mp4"
  },
  "seg": {
    "control_weight": 0.6,
    "control_path": "segmentation.mp4",
    "mask_path": "mask_inverted.mp4"
  }
}
```

#### 配方

<img src="./assets/omniverse_background_change_recipe.png" width="80%"/>

作为参考，下面是该 prompt：

```txt
In this video, a humanoid robot stands on a busy street sidewalk, approaching a red box on a small metal stand amid storefronts, street signs, and warm sunlight reflecting off nearby buildings.
```

### OV 背景生成结果

| 原始 | 已更换背景 |
|----------|----------|
| <video src="./assets/ov_rgb_video.mp4" controls width="300"></video> | <video src="./assets/omniverse_background_change.mp4" controls width="300"></video>  |

## 资源

1. [Cosmos Transfer 2.5 Model](https://github.com/nvidia-cosmos/cosmos-transfer2.5) - 模型权重与文档。
2. [控制模态概览](../../../../core_concepts/control_modalities/overview.md) - 各控制模态作用的概述。
3. 你可以使用这个 [Brev 实例](https://brev.nvidia.com/launchable/deploy?launchableID=env-36ZkptMd4kDDzFGMYN9lz9Ixnel) 体验这些配方，并将其自定义到你自己的视频上。它已完成全部预配置，并会自动提供所需算力，让你可以立即开始。

---

## 文档信息

**发布日期：** 2025 年 11 月 09 日

### 引用

如果你使用了此配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_multicontrol_recipes_with_2025,
  title={Multi-Control Recipes with Cosmos Transfer 2.5},
  author={Chang, Aiden and Santhosh, Akul},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/inference-real-augmentation/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Aiden Chang 与 Akul Santhosh（2025）。使用 Cosmos Transfer 2.5 的多控制配方。载于 *NVIDIA Cosmos Cookbook*。可访问：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/inference-real-augmentation/inference.html>
