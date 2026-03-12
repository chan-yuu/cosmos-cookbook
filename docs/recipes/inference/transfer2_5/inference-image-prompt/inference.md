# 使用 Cosmos Transfer 2.5 进行风格引导视频生成

> **作者：** [Fangyin Wei](https://weify627.github.io/) • [Aryaman Gupta](https://www.linkedin.com/in/aryamang/)
> **机构：** NVIDIA

## 概述

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| Cosmos Transfer 2.5 | 推理 | 使用图像参考进行风格引导视频生成 |

Cosmos Transfer 2.5 引入了一项强大的新能力：生成同时结合结构控制（edge/depth/segmentation）与参考图像风格引导的视频。这使用户能够在遵循精确运动和结构模式的同时，创建保持特定视觉美学的视频。

- [设置与系统要求](../inference-carla-sdg-augmentation/setup.md)

## 关键特性

- **图像引导的风格迁移**：使用任意图像作为视频生成的风格参考
- **多模态控制**：将 edge/depth/segmentation 控制与图像提示结合使用
- **灵活的风格应用**：控制参考图像对输出影响的强度
- **时间一致性**：在所有视频帧中保持连贯一致的风格

## 工作原理

1. **输入控制视频**：通过 edge、depth 或 segmentation 提供结构引导
2. **风格参考图像**：提供一张图像来定义期望视觉风格，而不改变输入控制视频引导的结构
3. **文本提示**：描述场景和期望输出
4. **模型处理**：Transfer 2.5 结合所有输入以生成风格化视频

> **注意：** 虽然 Cosmos Transfer 2.5 包含四个控制检查点（edge、blur、depth 和 segmentation），但图像提示功能仅支持 edge、depth 和 segmentation 控制。blur 控制与图像提示不兼容，因为它本身已包含颜色和风格引导，额外的图像提示会显得冗余且可能产生冲突。

## 数据集与设置

### 输入数据要求

对于风格引导视频生成，你需要：

- 一个控制视频（edge、depth 或 segmentation）
- 一张风格参考图像（JPEG/PNG）
- 一个描述期望输出的文本提示

### 数据结构

该流水线期望输入采用如下格式：

```
input_directory/
├── control_video.mp4    # Edge/depth/segmentation video
├── style_image.jpg      # Reference image for style
└── prompt.txt           # Text description
```

## 结果

以下两个示例展示了如何将不同环境风格应用到相同的 edge-controlled motion 上：

### 示例 1

**文本提示：**
> “镜头稳定地向前移动，模拟车辆沿街行驶时的视角。这种前进运动十分平滑，没有明显抖动或突兀的方向变化，从而持续展现城市景观。视频始终稳定聚焦于前方道路，随着镜头推进，建筑物逐渐向远处退去。整体氛围平静安宁，视野中没有行人或车辆，进一步突出了街道的空旷感。”

<table>
  <tr>
    <td colspan="3" align="center"><strong>输入</strong></td>
  </tr>
  <tr>
    <td align="center">Edge 控制<br><video controls autoplay muted loop width="300"><source src="./assets/example1_input_edge.mp4" type="video/mp4"></video></td>
    <td align="center">晴天风格<br><img src="./assets/example1_input_sunny.jpg" width="300"></td>
    <td align="center">日落风格<br><img src="./assets/example1_input_sunset.jpg" width="300"></td>
  </tr>
  <tr>
    <td colspan="3" align="center"><strong>输出</strong></td>
  </tr>
  <tr>
    <td align="center">基础生成<br><video controls autoplay muted loop width="300"><source src="./assets/example1_generation-from-edge.mp4" type="video/mp4"></video></td>
    <td align="center">应用晴天风格<br><video controls autoplay muted loop width="300"><source src="./assets/example1_generation-from-edge-sunny.mp4" type="video/mp4"></video></td>
    <td align="center">应用日落风格<br><video controls autoplay muted loop width="300"><source src="./assets/example1_generation-from-edge-sunset.mp4" type="video/mp4"></video></td>
  </tr>
</table>

### 示例 2

**文本提示：**
> “一段风景优美的海岸公路驾驶旅程徐徐展开。视频记录了沿多车道道路平稳、连续前进的过程，镜头位置仿佛来自一辆行驶在右侧车道上的车辆视角。道路右侧是一座高耸的绿色山体，山体的阴影覆盖了部分公路；左侧则豁然开朗，可以望见海景，远处海面位于一排低矮植被和人行道之外。前方有数辆车辆匀速行驶，其中包括两辆红色车辆。道路维护良好，白色车道线清晰可见，并有一道混凝土护栏将车道与右侧覆盖树木的山体分隔开来。道路左侧的电线杆和电力线与公路平行延伸，进一步丰富了场景基础设施。镜头保持静止，持续呈现道路及周边环境，突出了这段驾驶过程宁静且不中断的特质。”

<table>
  <tr>
    <td colspan="3" align="center"><strong>输入</strong></td>
  </tr>
  <tr>
    <td align="center">Edge 控制<br><video controls autoplay muted loop width="300"><source src="./assets/example2_input_edge.mp4" type="video/mp4"></video></td>
    <td align="center">更暗风格<br><img src="./assets/example2_input_darker.png" width="300"></td>
    <td align="center">更绿风格<br><img src="./assets/example2_input_greener.png" width="300"></td>
  </tr>
  <tr>
    <td colspan="3" align="center"><strong>输出</strong></td>
  </tr>
  <tr>
    <td align="center">基础生成<br><video controls autoplay muted loop width="300"><source src="./assets/example2_generation-from-edge.mp4" type="video/mp4"></video></td>
    <td align="center">应用更暗氛围<br><video controls autoplay muted loop width="300"><source src="./assets/example2_generation-from-edge-darker.mp4" type="video/mp4"></video></td>
    <td align="center">应用更绿色调<br><video controls autoplay muted loop width="300"><source src="./assets/example2_generation-from-edge-greener.mp4" type="video/mp4"></video></td>
  </tr>
</table>

### 关键观察

- **风格保留**：参考图像的调色板、光照和氛围能够成功迁移到生成视频中
- **结构保持**：Edge control 确保所有风格变化下的运动和物体边界保持一致
- **时间连贯性**：风格在整个视频序列中保持一致
- **灵活应用**：不同风格可以在保留底层运动的同时显著改变视频氛围

## 配置示例

### 基础风格引导生成

下面给出了一个 JSON 输入示例，用于运行已发布代码并生成示例 1 中展示的日落输出。

```json
{
    "name": "image_style",
    "prompt": "The camera moves steadily forward, simulating the perspective of a vehicle driving down the street. This forward motion is smooth, without any noticeable shaking or abrupt changes in direction, providing a continuous view of the urban landscape. The video maintains a consistent focus on the road ahead, with the buildings gradually receding into the distance as the camera progresses. The overall atmosphere is calm and quiet, with no pedestrians or vehicles in sight, emphasizing the emptiness of the street.",
    "video_path": "calm_street.mp4",
    "image_context_path": "sunset.jpg",
    "seed": 1,
    "edge": {
    }
}
```

### 配置参数

| 参数 | 说明 | 必填 |
|-----------|-------------|----------|
| `name` | 本次生成任务的标识符 | 是 |
| `prompt` | 对期望输出场景的文本描述 | 是 |
| `video_path` | 输入 RGB 视频路径（用于生成控制信号） | 是 |
| `image_context_path` | 风格参考图像路径（JPEG/PNG） | 是 |
| `seed` | 用于复现的随机种子 | 否 |
| `edge` / `depth` / `seg` | 控制模态配置（使用其中一种） | 是 |
| `control_weight` | 结构控制强度（0.0-1.0，默认：1.0） | 否 |

### 控制模态选项

你可以根据需要使用不同的控制类型。只有 **edge**、**depth** 和 **segmentation** 支持图像提示：

```json
// Edge control - preserves structure and shape
{ "edge": { "control_weight": 1.0 } }

// Depth control - maintains 3D spatial consistency
{ "depth": { "control_weight": 1.0 } }

// Segmentation control - enables semantic replacement
{ "seg": { "control_weight": 0.8 } }
```

## 最佳实践

### 风格图像选择

1. **光照一致性**：选择与目标场景光照相匹配的参考图像
2. **颜色协调性**：选择与内容互补的调色板图像
3. **质量很重要**：高分辨率参考图像能产生更好的风格迁移效果
4. **上下文相关性**：环境相似的图像效果最佳

### 参数调优

- **Control Weight**：在结构保留与风格灵活性之间取得平衡
  - 较高时更适合精确运动跟踪
  - 较低时更偏向艺术化表达

- **Guidance Scale**：影响对文本提示和参考图像的遵循程度
  - 更高的值：文本提示和参考图像的影响都会增强

## 应用场景

- **电影与动画**：在多个场景中应用一致的视觉风格
- **内容创作**：将视频转换为符合品牌美学的风格
- **艺术表达**：创建独特的视觉诠释
- **环境仿真**：生成不同光照/天气条件下的视频
- **风格一致性**：在一系列视频中保持视觉连贯性

## 故障排查

### 常见问题与解决方案

#### 风格应用不够明显

- 增加 guidance 参数
- 使用更有辨识度的参考图像
- 调整 prompt，强调风格元素

#### 运动连贯性丢失

- 提高 edge/depth/segmentation 的 `control_weight`
- 如果 guidance 过强，则适当降低
- 确保控制视频质量足够高

#### 颜色渗出或伪影

- 检查参考图像质量
- 降低 guidance scale
- 调整 guidance 与 control weight 的平衡

## 资源

- **[Cosmos Transfer 2.5 模型](https://github.com/nvidia-cosmos/cosmos-transfer2.5)** - 模型权重与文档
- **[控制模态指南](../../../../core_concepts/control_modalities/overview.md)** - 了解不同控制类型

---

## 文档信息

**发布日期：** 2025 年 12 月 20 日

### 引用

如果你使用了此配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_styleguided_video_generation_2025,
  title={Style-Guided Video Generation with Cosmos Transfer 2.5},
  author={Wei, Fangyin and Gupta, Aryaman},
  year={2025},
  month={December},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/inference-image-prompt/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Fangyin Wei 与 Aryaman Gupta（2025）。使用 Cosmos Transfer 2.5 进行风格引导视频生成。载于 *NVIDIA Cosmos Cookbook*。可访问：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/inference-image-prompt/inference.html>
