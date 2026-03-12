# Cosmos Transfer 2.5 用于模拟器视频的 Sim2Real

> **作者：** [Ryan Ji](https://www.linkedin.com/in/ryan-ji-a73300206/) • [Jingyi Jin](https://www.linkedin.com/in/jingyi-jin)
> **机构：** NVIDIA

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| [Cosmos Transfer 2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5) | 推理 | Sim to Real 数据增强 |

本教程演示如何使用 Cosmos Transfer 2.5 模型增强来自仿真的合成数据，将有限的模拟器输出转换为照片级真实数据集，同时减少扩展多样性所需的人工工作量。

- [设置与系统要求](setup.md)

## 为什么 Sim2Real 增强很重要

从模拟器创建多样化、照片级真实的训练数据存在显著挑战：

- **域差距**：虽然模拟器能够提供完美的真值和可控场景，但其合成外观会造成明显的域差距，从而限制基于模拟器数据训练的模型在真实环境部署时的表现。

- **可扩展性限制**：在模拟器中手动构建多样化场景需要大量工程投入和计算资源，导致扩展数据多样性的成本极高。

- **视觉真实感有限**：传统模拟器输出缺乏真实世界部署所需的照片级真实质量，因此需要额外的后处理或域适配技术。

Cosmos Transfer 2.5 可以将模拟器输出转换为照片级真实且多样的数据集，弥合 sim-to-real 差距，并提升下游模型在真实部署中的表现。

## 演示概览

这是一个使用 **Cosmos Transfer 2.5** 对合成数据进行 Sim2Real 增强的演示。本教程将逐步介绍如何把模拟器输出的合成结果转换为照片级真实且多样的数据集。借助 Cosmos Transfer 2.5 模型先进的生成能力，我们展示了如何在保持原始模拟器数据的结构完整性和语义信息的同时，弥合 sim-to-real 差距。

## 在模拟器中创建异常场景

### 手动创建场景的挑战

在模拟器中创建交通异常场景需要人工投入和技术专长：

- **地图设计**：必须手动构建或修改自定义道路网络，以支持特定异常场景。
- **交通设置**：每辆车都需要单独进行位置摆放、轨迹规划和行为脚本编写。
- **异常工程**：逆行行为需要精心编程，以确保其既真实又具有不安全特征。
- **相机配置**：必须布置并校准多个视角，以从相关角度捕捉异常情况
- **环境调节**：每种变化都需要手动调整光照、天气和一天中的时间设置。

这种高劳动强度的流程使得大规模创建多样化异常数据集的成本高得难以承受。

### 逆行驾驶场景

演示视频展示了一个在模拟器合成环境中捕获的关键交通安全场景：

> “该场景描绘了一个大型城市十字路口，路面上有醒目的黄色网格框，用于防止车辆堵塞路口；周围有多条车道，带有清晰的停止线、人行横道和道路两侧点缀着路灯、棕榈树与横幅的人行道。交通灯悬挂在上方，协调来自各个方向的车流，大多数车辆在红灯前有序排队或在绿灯时向前通行，而其中一辆车明显逆着正确车道方向行驶，形成了逆行交通异常。背景中可见高大且细节丰富的石材与玻璃建筑、带拱形入口的楼宇，以及现代与古典建筑风格的混合，还有清晰的标识和远处的街道活动，使整个环境呈现出真实、繁忙的都市氛围。”

这个复杂的城市场景展示了以下特点：

- **交通异常**：一辆车在其他交通基本有序的情况下逆着正确车道方向行驶。
- **丰富的城市语境**：细节丰富的路口、交通基础设施、建筑物和城市元素。
- **合成外观**：尽管场景构成细致，但其典型的渲染风格仍暴露了模拟器来源。
- **安全关键行为**：逆行车辆制造了危险情境，自主系统必须检测到它。

### 模拟器提供的真值

模拟器为每一帧提供了完整的真值数据：

<div style="display: flex; flex-wrap: wrap; gap: 20px; justify-content: space-between;">
  <div style="flex: 1 1 45%; min-width: 300px;">
    <strong>RGB 视频</strong>：展示交通场景的原始合成渲染
    <video controls width="100%" aria-label="展示交通路口中逆行车辆异常的 RGB 视频">
      <source src="./assets/simulator_rgb_input.mp4" type="video/mp4">
      你的浏览器不支持 video 标签。
    </video>
  </div>
  <div style="flex: 1 1 45%; min-width: 300px;">
    <strong>深度图</strong>：场景中每个像素的精确距离信息
    <video controls width="100%" aria-label="展示交通场景距离信息的深度图视频">
      <source src="./assets/simulator_depth.mp4" type="video/mp4">
      你的浏览器不支持 video 标签。
    </video>
  </div>
</div>

<div style="display: flex; flex-wrap: wrap; gap: 20px; justify-content: space-between; margin-top: 20px;">
  <div style="flex: 1 1 45%; min-width: 300px;">
    <strong>边缘检测</strong>：所有物体和道路基础设施的几何边界
    <video controls width="100%" aria-label="展示物体与基础设施几何边界的边缘检测视频">
      <source src="./assets/simulator_edge.mp4" type="video/mp4">
      你的浏览器不支持 video 标签。
    </video>
  </div>
  <div style="flex: 1 1 45%; min-width: 300px;">
    <strong>语义分割</strong>：对车辆、道路、人行道和其他场景元素的像素级标签
    <video controls width="100%" aria-label="展示包含车辆和道路等已标注场景元素的语义分割视频">
      <source src="./assets/simulator_segmentation.mp4" type="video/mp4">
      你的浏览器不支持 video 标签。
    </video>
  </div>
</div>

这些控制信号构成了 Cosmos Transfer 2.5 模型进行照片级真实增强的基础，同时确保异常行为得到保留。

## 面向照片级真实增强的 Prompt Engineering

### 通过策略性提示改造合成数据

Cosmos Transfer 2.5 利用精心设计的提示，将模拟器输出的合成结果转换为照片级真实场景。成功增强的关键在于三个核心组成部分：

1. **Positive Prompts**：详细描述，引导模型在保留异常行为的同时朝着照片级真实效果生成
2. **Negative Prompts**：用于防止不真实伪影并保持结构完整性的约束
3. **模型自身能力**：模型对真实世界物理和光照的理解，用于增强真实感

### 使用物理 AI 模型 Cosmos Reason 1 进行场景理解

我们的提示工程流水线利用 Cosmos Reason 1——一个在 AV 与机器人数据上进行密集训练、具备卓越物理场景理解能力的模型——通过两阶段方法将特定变化嵌入场景描述中：

#### 阶段 1：全局场景描述

我们使用 Cosmos Reason 1-7B 生成全面的说明文字，以基于物理 AI 专长捕捉所有场景元素：

- 交通基础设施（道路、路口、交通灯）
- 车辆及其行为（包括异常）
- 环境上下文（建筑、植被、城市特征）
- 当前条件（一天中的时间、天气、能见度）

#### 阶段 2：特定变化的增强

然后，使用 Llama-3.1-8B-Instruct 对全局描述进行处理，在保留核心场景结构的同时注入特定增强关键词。向该 LLM 提供的内容包括：

- 来自 Cosmos Reason 1-7B 的原始全局描述
- 目标变化关键词（例如 “night”“snow falling”“puddles”）
- 仅对相关方面进行真实修改的指令

这种方法确保异常行为和场景结构保持不变，而只有期望的视觉属性被转换。Cosmos Reason 1 模型在自动驾驶和机器人数据上的专项训练，使其能够准确理解空间关系、车辆动力学和交通场景，这对于在增强过程中维持真值完整性至关重要。

### 增强类别与提示设计

为了最大化数据多样性，我们采用了 18 种不同的增强类型，并将其组织为三个高级类别：

| 类别 | 条件与增强思路 |
|----------|--------------------------------|
| **环境光照** | **Sunrise**：温暖的晨光、长阴影、东方橙粉色天空<br>**Sunset**：温暖的傍晚光线、长阴影、西方橙粉色天空<br>**Twilight**：冷蓝色调、低环境光、漫射背景<br>**Mid-morning**：清晰日光、平衡阴影、清晰纹理<br>**Afternoon**：中性日光、自然阴影、明亮曝光<br>**Zenith**：强烈顶光、短阴影、强对比<br>**Golden hour**：柔和暖色调、拉长的阴影、更强的景深感<br>**Blue hour**：深蓝色调、弱光、柔和环境照明<br>**Night**：低照度、高对比、车灯/前灯辉光、路灯 |
| **天气** | **Clear Sky**：高能见度、明显阴影、浅蓝色渐变天空<br>**Overcast**：平坦的漫射光、灰白天空、阴影减少<br>**Snow Falling**：飘雪覆盖、冷白平衡、低饱和色调<br>**Raining**：雨丝、积水反射、湿路面光泽<br>**Fog**：基于深度的雾霾、白/灰叠加、远处遮蔽 |
| **路面状况** | **Dry road**：干净沥青、可见车道线、一致反射率<br>**Snow on ground**：白色地表覆盖、轮胎痕迹、积雪堆<br>**Sand on ground**：浅褐色表面、颗粒纹理、尘雾<br>**Puddles**：积水、镜面反射、涟漪 |

### 保留异常行为

为每种增强编写提示时，会特别关注以下准则：

- **保持可见性**：确保逆行车辆在所有条件下都能被检测到。
- **保留空间关系**：保持相对位置和轨迹不变。
- **增强真实感**：在不遮蔽安全关键特征的前提下加入照片级真实元素

雪地增强的提示结构示例：

**Positive Prompt：**

```
"The video depicts a bustling urban intersection during daytime, with clear skies and ample sunlight illuminating the scene, casting a pale light on the snow-covered ground. The environment is characterized by modern buildings with large windows and classical architectural elements, suggesting a cityscape that blends contemporary and historical design. The intersection is marked with yellow grid lines on the road, now partially obscured by a layer of snow, indicating pedestrian crossing areas. Several vehicles are present, including a red car in the foreground moving diagonally across the intersection, leaving behind dark tire tracks in the snow, a black SUV turning right, and other cars in various colors such as blue, white, and green, all navigating through the busy street, their tires creating visible grooves in the snow. Traffic lights are visible at the intersection, with some showing red signals, and snow piles are accumulated along road edges and curbs."
```

**Negative Prompt：**

```
"The video captures a game playing, with bad crappy graphics and cartoonish frames. It represents a recording of old outdated games. The lighting looks very fake. The textures are very raw and basic. The geometries are very primitive. The images are very pixelated and of poor CG quality. There are many subtitles in the footage. Overall, the video is unrealistic at all."
```

这种提示工程方法确保了以下几点：

- 异常（车辆斜向移动/逆行）始终清晰可见。
- 雪效会以真实方式应用，而不会遮挡关键细节。
- 合成外观被转换为照片级真实质量。
- 来自模拟器的真值信息得到保留。

## Cosmos Transfer 2.5 输出示例

### 比较控制模型的影响

为了展示 Cosmos Transfer 2.5 的多样能力，我们展示了跨三类增强类型的不同控制配置输出。每种控制模型（depth、segmentation 和 edge）都会在保留关键异常行为的同时，生成不同的照片级真实变换结果。

### 选定的增强示例

为了展示 Cosmos Transfer 2.5 的多样能力，我们给出了五个具有代表性的增强示例，展示不同环境条件和控制策略：

#### 1. 夜间增强（使用 Depth Control）

<video controls width="100%" style="max-width: 800px;" aria-label="展示带有路灯和车辆前灯的夜间增强交通场景">
  <source src="./assets/night_depth_output.mp4" type="video/mp4">
  你的浏览器不支持 video 标签。
</video>
*Depth control 在创建具有路灯和车灯效果的戏剧化夜间光照时，能保留空间关系*

#### 2. 上午中段增强（使用 Segmentation Control）

<video controls width="100%" style="max-width: 800px;" aria-label="展示明亮日间条件下交通场景的上午增强效果">
  <source src="./assets/mid_morning_seg_output.mp4" type="video/mp4">
  你的浏览器不支持 video 标签。
</video>
*Segmentation control 可在明亮自然的日光条件下确保物体边界一致*

#### 3. 降雪增强（使用 Edge 控制）

<video controls width="100%" style="max-width: 800px;" aria-label="展示大雪天气交通场景的降雪增强效果">
  <source src="./assets/snow_falling_edge_output.mp4" type="video/mp4">
  你的浏览器不支持 video 标签。
</video>
*Edge control 能在大雪中保持几何清晰度，保留关键道路边界*

#### 4. 雾天增强（使用 Depth Control）

<video controls width="100%" style="max-width: 800px;" aria-label="展示因雾而能见度降低的交通场景雾天增强效果">
  <source src="./assets/fog_depth_output.mp4" type="video/mp4">
  你的浏览器不支持 video 标签。
</video>
*基于深度的雾效会自然遮蔽远处物体，同时保持近处车辆和道路特征可见*

#### 5. 黄昏增强（使用 Depth Control）

<video controls width="100%" style="max-width: 800px;" aria-label="展示黄昏过渡光照条件下交通场景的暮光增强效果">
  <source src="./assets/twilight_depth_output.mp4" type="video/mp4">
  你的浏览器不支持 video 标签。
</video>
*带有深蓝色调与过渡光照的黄昏条件，呈现出暮色下具有挑战性的可见性*

### 关键观察

- **Depth Control**：最适合保持 3D 空间一致性和真实遮挡关系
- **Segmentation Control**：最适合保留语义边界和针对特定物体的变换
- **Edge 控制**：非常适合保留结构细节和几何精度

所有输出都成功将合成外观转换为照片级真实场景，同时确保逆行车辆异常在训练稳健安全系统时仍清晰可检测。

## Cosmos Transfer 2.5 中的控制参数

### 配置结构

Cosmos Transfer 2.5 提供灵活的控制参数，可对增强过程进行微调。配置决定了输出在实现照片级真实变换的同时，应多大程度遵循来自模拟器的结构信息。

### 基本配置格式

配置文件采用 JSON 结构，用于指定输入路径、输出目录和控制参数：

```json
{
    "prompt_path": "assets/car_example/car_prompt.json",
    "output_dir": "outputs/car_edge",
    "video_path": "assets/car_example/car_input.mp4",
    "control_weight": 1.0,
    "edge": {
        "control_path": "assets/car_example/edge/car_edge.mp4"
    }
}
```

### 控制类型

若要使用不同的控制类型，只需将配置中的 `"edge"` 替换为表示分割控制的 `"seg"` 或表示深度控制的 `"depth"`，同时相应更新 `control_path`。

## 保持真值完整性

### 保留关键异常行为

Sim2Real 增强的一个基本要求是保持真值数据的完整性。Cosmos Transfer 2.5 擅长在改造合成视觉效果的同时，保留使这些数据对训练有价值的精确行为和轨迹。

### 异常保留的可视化验证

生成的增强视频在所有环境条件下都清晰保留了逆行行为：

#### 异常行为对比

| 原始模拟器 | 增强输出 | 异常状态 |
|-------------------|------------------|----------------|
| ![原始异常](./assets/original_anomaly_trajectory.gif) | ![增强后异常](./assets/augmented_anomaly_trajectory.gif) | ✓ 逆行车辆清晰可见且可被检测 |

### 真值保留特性

Cosmos Transfer 2.5 确保模拟器提供的标注仍然有效：

- **车辆轨迹**：逐帧位置与模拟器中完全一致
- **Bounding Boxes**：增强后目标检测标注仍保持准确
- **语义标签**：车辆分类和道路标线保持原始标签
- **时间一致性**：异常出现的时机和持续时间不变

### 质量保证结果

我们的验证显示如下：

- 在全部 18 种增强类型中，**100% 保留异常**
- 与原始模拟器数据相比，**bounding-box 对齐达到像素级精度**
- 使用相同异常检测模型时，**检测率保持一致**
- **增强后的视觉清晰度**使人工审查者更容易发现异常

由于真值完整性得到保留，基于 Cosmos Transfer 2.5 增强数据训练的模型，能够在保持模拟器中定义的安全关键行为完全不变的前提下，从照片级真实图像中学习。

## 扩展数据多样性

### 从单一场景到完整数据集

Cosmos Transfer 2.5 改变了合成数据生成的成本结构。我们从单个模拟器场景出发，生成了 18 种不同的增强变体，在无需额外模拟器人工操作的情况下显著扩展了训练数据多样性。

### 增强矩阵结果

基于原始的逆行驾驶场景，我们使用单一控制配置生成了完整的增强矩阵。

#### 完整增强网格

<div style="max-width: 95%; margin: 0 auto;">
<img src="./assets/augmentation_matrix_grid.gif" alt="Augmentation Matrix" style="width: 100%; height: auto; display: block;">
</div>

该矩阵展示了如下组织的全部 18 种变化：

- **9 种环境光照条件**
- **5 种天气条件**
- **4 种路面状况**

这个完整网格展示了使用一种控制类型、从单个模拟器场景能够实现的全部照片级真实增强范围。每种变化都在改变视觉外观的同时保留了关键的逆行行为。根据具体用例需求，也可以基于不同控制配置（depth、edge 或 segmentation）生成类似网格。

### 成本收益分析

下表展示了与传统模拟器方法相比，使用 Cosmos Transfer 2.5 生成多样化训练场景的显著优势。

| 方面 | 传统模拟器 | Cosmos Transfer 2.5 |
|--------|----------------------|---------------------|
| **设置时间** | 18 个场景 × 手动设置 = 数周工程投入 | 1 个基础场景 + 18 个提示 = 数小时生成完整数据集 |
| **处理方式** | 18 个场景 × 渲染时间 = 大量计算成本 | 并行处理 = 同时生成全部变体 |
| **灵活性** | 额外变体的灵活性有限 | 无限灵活性 = 易于添加新的增强类型 |

### 数据质量验证

每个增强输出都保持了以下特性：

- 原始异常行为（即逆行车辆轨迹）
- 来自模拟器的一致 ground-truth 标注
- 适合真实世界部署的照片级真实外观
- 有助于稳健模型训练的多样环境条件

这种可扩展方法使团队能够创建全面的训练数据集，从而让 AI 系统为其可能遇到的全部真实世界条件做好准备。

## 结论

本教程展示了 Cosmos Transfer 2.5 如何革新物理 AI 应用中的合成数据生成。通过将基础模拟器输出转换为照片级真实且多样的数据集，它使构建面向自动驾驶车辆和机器人的稳健世界模型成为可能。

这个逆行驾驶场景展示了以下成果：

- **18 倍数据扩增**：从单个模拟器场景扩展到 18 种照片级真实变体，覆盖不同光照、天气和路面条件
- **保留真值**：在所有增强中对异常行为和模拟器标注实现 100% 保留
- **可用于生产的质量**：输出具备可训练安全关键 AI 系统的照片级真实质量
- **灵活的控制选项**：提供 depth、segmentation 和 edge 控制，以适配不同用例

### 对物理 AI 开发的影响

Cosmos Transfer 2.5 解决了物理 AI 中的关键挑战：

1. **成本效率**：消除每个场景数周的模拟器手工工程投入
2. **安全性**：无需真实世界风险即可在危险场景上进行训练
3. **可扩展性**：让更多人能够获得多样化、高质量训练数据
4. **可靠性**：为安全关键应用保持真值完整性

通过结合先进的生成式 AI 与来自 Cosmos Reason 1 的物理场景理解，该流水线使企业和研究人员能够构建更稳健的物理 AI 系统。照片级真实质量与真值保留的结合，使其对自动驾驶开发尤其有价值，因为该领域同时高度依赖视觉保真度和行为准确性。

有关实现细节和更多用例，请参阅[设置指南](setup.md)，并探索 Cosmos 生态中的更多示例。

---

## 文档信息

**发布日期：** 2025 年 10 月 09 日

### 引用

如果你使用了此配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_cosmos_transfer_25_2025,
  title={Cosmos Transfer 2.5 Sim2Real for Simulator Videos},
  author={Ji, Ryan and Jin, Jingyi},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/inference-carla-sdg-augmentation/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Ryan Ji 与 Jingyi Jin（2025）。Cosmos Transfer 2.5 用于模拟器视频的 Sim2Real。载于 *NVIDIA Cosmos Cookbook*。可访问：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/inference-carla-sdg-augmentation/inference.html>
