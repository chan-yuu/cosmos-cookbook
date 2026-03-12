# 面向 AV 视频描述和 VQA 的 SFT

> **作者：**
> [DeLesley Hutchins](https://www.linkedin.com/in/delesley-hutchins-ba48532b) • [Jingyi Jin](https://www.linkedin.com/in/jingyi-jin/) • [Amol Fasale](https://www.linkedin.com/in/amolfasale/) (**NVIDIA**) •
> [Joseph Wang](https://www.linkedin.com/in/josephwangphd/) (**Uber**)

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Reason 1](https://github.com/nvidia-cosmos/cosmos-reason1) | 后训练 | AV 视频描述和视觉问答 |

## 概述

自动驾驶车辆必须在各种不同条件下运行。这些条件包括不同类型的天气、一天中的不同时间、不同类型的道路、路况、驾驶环境和交通状况。自动驾驶车辆还必须能够识别并响应交通信号、张贴的交通标志和道路标线。此外，它们还必须能够识别危及人类生命安全的情况，例如骑行者和行人的存在。

从 dashcam 收集大量驾驶视频数据相对容易，但这些数据最初是未标注的。为了使用视频数据，通常需要高质量的 labels 或 captions；这些标签可以让视频片段被组织成可搜索数据库，从而能够定位有趣的驾驶场景，用于训练或测试。人工标注质量高，但成本昂贵。像 Cosmos Reason 这样的视频语言模型则可以作为 auto-labeler 或 auto-captioner 使用。

本配方说明如何微调 Cosmos Reason，使其能够为视频数据生成高质量、领域特定的 captions 和 labels。基础版 Reason 模型本身就能生成 captions，但我们表明，通过在人工标注数据集上微调，可以进一步提高 captions 的质量。

在人工标注上进行微调，也是提升模型在领域特定任务上 visual question answering (VQA) 能力的一种好方法。本配方中使用的许多具体标注也可以被解释为对模型提出的问题，例如“天气条件是什么？”

本配方的基本步骤如下：

- **数据**：收集带有人类标注的视频 clips 初始数据集。
- **SFT**：在该数据集上对模型进行监督微调。
- **评估**：使用另一个 LLM 作为 judge 来评估结果质量。

## 数据

在本配方中，我们使用 NVIDIA 的内部数据集，该数据集由短视频 clips 组成，每个 clip 长度在 5 到 10 秒之间。这些 clips 被交给一组人工标注员，他们会回答关于 clip 的一系列问题。

下面是一个示例视频 clip：

<video controls width="1280">
  <source src="assets/01a6de83-ac02-4689-a901-f0d1dfc0672d.small.mp4" type="video/mp4">
</video>

下面是该 clip 的人工标注，采用 json 格式：

