# 用于智能交通系统 (ITS) 图像天气增强的 Cosmos Transfer 1

> **Authors:** [Reihaneh Entezari](https://www.linkedin.com/in/reihanehentezari/) • [Charul Verma](https://www.linkedin.com/in/charul-verma-6bb778172/) • [Arihant Jain](https://www.linkedin.com/in/arihant-jain-5955046b/) • [Dharshi Devendran](https://www.linkedin.com/in/dharshidevendran/) • [Ratnesh Kumar](https://www.linkedin.com/in/rkumar1729/)
> **Organization:** NVIDIA

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Transfer 1](https://github.com/nvidia-cosmos/cosmos-transfer1) | Inference | Data augmentation |

本教程演示如何使用 Cosmos Transfer 1 模型进行合成数据生成（SDG），以增强数据并提升下游计算机视觉（CV）或视觉语言模型（VLM）算法的准确性。

- [安装与系统要求](setup.md)

## 为什么天气增强很重要

在恶劣天气条件下采集智能交通系统（ITS）数据既耗时又困难，而为预先录制的数据集加入基于天气的多样性往往成本高昂，甚至无法实现。数据稀缺会导致计算机视觉模型在真实世界恶劣天气场景中的表现不佳。我们可以通过使用 Cosmos Transfer 1 执行天气增强来弥补这一关键缺口，从现有的晴天数据集中生成具有天气多样性的合成训练数据，从而显著提升下游 ITS 目标检测模型在各种天气条件下的鲁棒性和准确性。

## 演示概览

这是一个使用 Cosmos Transfer 1 对 ITS 图像进行天气增强的演示。为展示其效果，本教程将逐步讲解如何使用 Cosmos Transfer 1 对 ITS 图像进行天气增强，以提升下游 ITS 目标检测器 RT-DETR 模型的表现。

## Cosmos Transfer 1 流程组件

### ITS 图像/视频示例输入

从晴朗白天的高速公路场景开始，作为增强的基础输入。

![Input image](assets/input.jpg)

> **Note**: 如果你处理的只有图像，在继续执行后续步骤并使用 Cosmos Transfer 1 之前，需要先将重复图像帧制作成视频。

### 使用 VLM 进行视频描述生成

首先，使用 VLM 作为描述生成器，为输入图像生成详细描述：

> “该视频展示了一个繁忙的高速公路场景，从树木和车辆投下的长阴影来看，时间似乎是傍晚时分。天空晴朗蔚蓝，表明天气良好且没有可见云层。高速公路为多车道，双向车流通行。道路上有多辆汽车，颜色和车型各异，说明其中混有多种私人车辆。高速公路两侧环境由茂密的绿色植被组成，边缘排列着树木。背景中可以看到丘陵或低山，为场景增加了纵深感。整体氛围平静而活跃，符合工作日通勤时段的典型特征。”

> **Note**: 我们使用 Qwen 2.5 VL 进行描述生成，输入提示词如下：

> *Describe this video in detail and please include weather or time of the day detail at the start.*

### 使用 LLM 为雨夜条件增强提示词

接下来，使用 LLM 根据目标天气条件（雨夜）增强描述：

> “该视频展示了一个繁忙的高速公路雨夜场景，笼罩在深沉的黑暗之中。天空被遮蔽，仅有零星的车灯和昏暗的路灯可见，在被雨水浸湿的道路上投下微弱的反光。高速公路为多车道，双向车流通行，周围景观几乎难以辨认，只能依稀看出高速公路两侧有茂密的绿植，背景中还有丘陵或低山，但这些特征因黑暗而仅能隐约看见。道路上有多辆颜色和车型各异的汽车，它们的车灯照亮了湿漉漉的路面，路面上的积水和细小水流微微闪烁。整体氛围平静而活跃，是典型的夜间通勤场景。”

NOTE: 我们使用 Llama 3.1，并采用以下输入提示词进行天气增强：

> *I have the following caption describing a video captured by a CCTV camera. I want to adapt this caption so that it accurately reflects the following environmental condition. The updated caption will be used as a prompt for a video generation model.*
>
>*Please update the original caption minimally and realistically — do not add any details that are not already implied or explicitly stated. Avoid hallucinations, assumptions, or dramatic interpretations. Do not add events, objects, or actions that do not exist in the original caption. Simply modify the scene description to reflect the given weather or lighting condition while preserving the factual content and structure.*
>
>*If the environmental condition indicates **daytime**, remove any mention of headlights, artificial lighting, or shadows caused by artificial sources.*
> *If the condition implies **nighttime or low visibility** (e.g., fog, heavy rain, snow at night), you may include realistic visual effects such as headlights, reflections, or dim street lighting — but only if consistent with the original caption.*
>
> *Format the output as a single paragraph enclosed within angle brackets `<...>`. Do not include any other text.*
>
> *Original Caption:*
> *{video_caption}*
>
> *Environmental Condition to Reflect:*
> *{condition}*

雨夜的天气条件描述如下：

> *Rainy night shrouded in deep darkness, with only scattered headlights and dim streetlights casting faint reflections on the rain-soaked road, where puddles and thin streams of water shimmer faintly.*

### Cosmos Transfer 1 在雨夜条件下的输出图像

使用增强后的提示词，Cosmos Transfer 1 可以在保留原始场景结构元素的同时，生成逼真的雨夜场景。

![Rainy night](assets/rainy_night.jpg)

> **Note**: 我们选择了 Cosmos Transfer 1 输出视频的中间帧。

## Cosmos Transfer 1 中的控制参数

总体而言，在运行 Cosmos Transfer 1 时可以控制 vis、edge、segmentation 和 depth。然而，实验表明，当仅控制 **segmentation 和 depth** 时，生成图像最适合用于白天/天气增强，尤其是在生成夜间场景时。

### 推荐的控制配置

推荐的控制配置文件如下：

```json
{
    "input_video_path": "assets/example1_input_video.mp4",
    "negative_prompt": "The video shows an unrealistic traffic scene with floating or jittery cars that ignore physics, sliding without friction, turning sharply without steering, or vanishing mid-motion. Vehicles overlap unnaturally, lack weight or inertia, and don't align with the road. Road markings are inconsistent or missing, and lanes appear distorted. Lighting is flickering and fake, while backgrounds look melted or warped. Pedestrians and traffic signals are misshapen, duplicated, or misplaced. Overall, the scene feels chaotic, lacks depth, structure, and visual coherence.",
    "guidance": 8.0,
    "sigma_max": 90.0,
    "depth": {
        "control_weight": 0.9
    },
    "seg": {
        "control_weight": 0.9
    }
}
```

> **Note**: 使用了 0.9 的控制权重，以便输出图像更好地贴合输入图像。

### 移除 Vis 和 Edge 控制

为了说明在使用 Cosmos Transfer 1 时为什么要移除 vis 和 edge 控制，我们基于同一张输入图像生成了两张雨夜图像：

1. **使用全部控制**（vis、edge、seg、depth）：无法按预期生成夜景。
2. **仅使用 seg 和 depth 控制**：生成图像达到了期望的黑暗程度。

#### Cosmos Transfer 1 生成图像 - 全部控制（vis、edge、depth、seg 的 control weights 为 0.9）

![Rainy night with all controls](assets/rainy_night_all_09.jpg)

#### Cosmos Transfer 1 生成图像 - 仅使用 Depth 和 Seg（control weights 为 0.9）

![Rainy night with depth and segmentation controls](assets/rainy_night.jpg)

如上所示，使用全部控制（vis、edge、depth、seg）无法生成合适的夜景，而仅使用 depth 和 seg 控制则能生成符合预期的黑暗图像。

## 训练下游 ITS 检测器

为了说明 Cosmos Transfer 1 天气增强对下游 ITS 检测器的影响，我们分别使用有无 Cosmos Transfer 1 增强图像（采用所有可能的天气/光照条件）训练了 RT-DETR 检测器。我们在三个公开 KPI 上评估了训练后的模型。

## 结果

我们使用约 ~12k 张真实 ITS 图像和约 ~84K 张由 Cosmos Transfer 1 通过本教程所述流程生成的天气增强图像进行了实验。以下是三个 KPI 的结果：

## ACDC 数据集

ACDC 数据集是一个与智能交通系统（ITS）相关的数据集，包含雪、雾、雨和夜间等不同天气条件。该数据集约包含 ~2k 张图像，可在此处获取：<https://acdc.vision.ee.ethz.ch/>

以下是在所有天气条件下，数据集中最常见目标（car、person、bicycle）的 AP50 结果：

![Result plot ACDC](assets/acdc_plots.png)
如上所示，蓝色曲线（来自使用 Cosmos Transfer 1 增强图像训练的检测器）在所有天气/光照条件和目标类别上都持续高于红色曲线的 AP50。

## SUTD 数据集

SUTD 数据集是另一个 ITS 相关数据集，天气条件更加多样，包括雪、雾、雨、夜间、多云和晴天。该数据集约包含 ~10k 张图像，可在此处获取：<https://sutdcv.github.io/SUTD-TrafficQA/#/download>

以下是在所有天气条件下，数据集中最常见目标（car、person、bicycle）的 AP50 结果：

![Result plot SUTD](assets/sutd_plots.png)
如上所示，蓝色曲线（来自使用 Cosmos Transfer 1 增强图像训练的检测器）在所有天气/光照条件和目标类别上都持续高于红色曲线的 AP50。

## DAWN 数据集

DAWN 数据集也是一个 ITS 相关数据集，包含雪、雾、雨和沙尘等不同天气条件。该数据集约包含 ~1k 张图像，可在此处获取：<https://www.kaggle.com/datasets/shuvoalok/dawn-dataset>

以下是在所有天气条件下，数据集中最常见目标（car、person）的 AP50 结果：

![Result plot DAWN](assets/dawn_plots.png)
如上所示，蓝色曲线（来自使用 Cosmos Transfer 1 增强图像训练的检测器）在所有天气/光照条件和目标类别上都持续高于红色曲线的 AP50。

## 结论

本教程展示了 Cosmos Transfer 1 如何有效地为 ITS 数据集增加具有挑战性的天气条件，从而提升下游目标检测模型的性能。以下是关键要点：

- 仅使用 segmentation 和 depth 控制可获得最佳天气增强效果。
- 合理的提示词工程对于生成逼真的天气条件至关重要。
- 合成增强能显著提升模型在稀有天气条件下的表现。

有关实现和训练配置的更多细节，请参阅本仓库中配套的设置与配置文件。

---

## 文档信息

**Publication Date:** October 09, 2025

### 引用

如果你使用了此配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_cosmos_transfer_1_2025,
  title={Cosmos Transfer 1 Weather Augmentation for Intelligent Transportation System (ITS) Images},
  author={Entezari, Reihaneh and Verma, Charul and Jain, Arihant and Devendran, Dharshi and Kumar, Ratnesh},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer1/inference-its-weather-augmentation/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**Suggested text citation:**

> Reihaneh Entezari, Charul Verma, Arihant Jain, Dharshi Devendran, & Ratnesh Kumar (2025). Cosmos Transfer 1 Weather Augmentation for Intelligent Transportation System (ITS) Images. In *NVIDIA Cosmos Cookbook*. Accessible at <https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer1/inference-its-weather-augmentation/inference.html>
