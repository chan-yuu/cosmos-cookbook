# Cosmos Predict 2 Text2Image 用于智能交通系统（ITS）图像

> **作者：** [Charul Verma](https://www.linkedin.com/in/charul-verma-6bb778172/)  • [Reihaneh Entezari](https://www.linkedin.com/in/reihanehentezari/) • [Arihant Jain](https://www.linkedin.com/in/arihant-jain-5955046b/) • [Dharshi Devendran](https://www.linkedin.com/in/dharshidevendran/) • [Ratnesh Kumar](https://www.linkedin.com/in/rkumar1729/)
> **机构：** NVIDIA

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| [Cosmos Predict 2](https://github.com/nvidia-cosmos/cosmos-predict2) | 推理 | 合成数据生成 |

本教程演示如何使用 [Cosmos Predict 2](https://github.com/nvidia-cosmos/cosmos-predict2) Text2Image 模型进行合成数据生成（Synthetic Data Generation，SDG），以提升下游计算机视觉（Computer Vision，CV）或视觉语言模型（Vision-Language Model，VLM）算法的准确性。

- [设置与系统要求](setup.md)

## 为什么 ITS 合成数据生成很重要

大规模采集高质量 ITS 数据成本高、速度慢，而且通常并不完整：

- 多样化场景难以按需采集（如城市道路与高速公路、各类交叉路口）。
- 长尾类别代表性不足：路牌、骑行者、行人和摩托车出现频率较低，导致类别不平衡。
- 一天中不同时间、天气和相机角度带来的变化会引入领域偏移，难以被全面覆盖。

Cosmos Predict 2 支持有针对性的合成数据生成，从而有策略地填补这些空白。借助 text-to-image 控制，我们可以：

- 生成更加均衡的数据集，在保持场景真实感的同时，对低频类别（例如特定路牌、自行车）进行上采样。
- 系统性地遍历相机视角（top-down、dashboard）、光照（清晨/白天/黄昏/夜晚）、天气和场景布局，以提升泛化能力。

最终得到的是一组经过精心整理的场景组合，它既可补充真实数据、提升罕见案例的覆盖率，又能缩小领域差距，从而最终提升生产环境中下游 CV/VLM 的性能与鲁棒性。

## 演示概览

这是一个使用 **Cosmos Predict 2** 生成 ITS 图像的演示。为了展示其效果，本教程将逐步讲解 Cosmos Predict 2 的 ITS 图像生成流程，以及它如何改进下游 ITS 目标检测器 RT-DETR 模型。

## Cosmos Predict 2 流水线组件

## 架构

![输入图像](assets/architecture.png)

### 组件说明

- VLM Captioner：从示例 ITS 图像中生成忠实、详细的描述，作为生成过程的起点。
- LLM Prompt Augmenter：在严格的真实感约束下，注入目标实体和变化因素（视角、天气）。
- Cosmos Predict 2 Text‑to‑Image：根据提示词生成高质量 ITS 图像。
- Train RT‑DETR：对检测器进行微调。
- Evaluate on KPIs：衡量在 ACDC、SUTD、DAWN 上的改进效果（例如按类别和天气统计的 AP50）。

### ITS 输入图像示例

先从一张 ITS 图像开始，以此作为生成描述的参考。

![输入图像](assets/input.jpg)

### 使用 VLM 进行图像描述

首先，使用 VLM 作为 captioner，为输入图像生成详细描述：

> “车辆内部视角下的夜间街景中，一位摩托车骑手正在车流中穿行，汽车和路灯照亮了湿漉漉的道路。”

注意：我们使用 Qwen 2.5 VL 进行描述生成，输入提示词如下：

> 给定这张图像，请生成一段描述，只描述图像中满足以下所有条件的内容：
>
> 1. 符合物理规律。
> 2. 物体的尺度和位置相对于彼此及环境都是现实合理的。
> 3. 具有视觉一致性（即所有物体都属于该环境，且彼此自然共存）。
> 4. 物体具有正确的纹理/几何形态。
> 5. 确保描述类似真实照片的说明。避免使用 'scene'、'depicts' 或 'imagines'。

### 使用 LLM 引导描述增强以引入目标对象（例如自行车）

接下来，使用 LLM 对描述进行增强，以引入目标对象（自行车）：

> “车辆内部视角下的夜间街景中，一位摩托车骑手正在车流中穿行，汽车和路灯照亮了湿漉漉的道路。人行道附近的一根路灯杆旁倚着一辆自行车，车主不见踪影。远处，一名骑行者正骑着自行车朝相反方向驶来，其反光装备映照着迎面车辆的灯光。与此同时，一位自行车快递员在车流中穿梭，避开一辆正拐入停车场的汽车，而停车场入口附近的车架上还锁着另一辆自行车。”

注意：我们使用 Llama 3.1，并采用以下输入提示词进行以对象为中心的增强：

> """给定一个描述场景的基础图像描述。仅在场景在逻辑上能够容纳这些对象时，通过加入真实且符合上下文变化的人、bicycle 和 car（包括 bike、motorbike、truck 或 bus 等同义词）实例来增强该描述。修改后的描述将用于生成图像。
>
> 严格规则：
>
> 1. 除天气外，精确保留原始环境与场景设定。如果原始场景不是道路或交通场景，不要将其转换成此类场景。
> 2. 如果场景位于室内、无可见道路的校园区域、人行道上，或其他不适合车辆出现的环境中，DO NOT 添加任何车辆或道路元素。
> 3. 仅当与原始场景类型相符时，才使用给定的相机角度和视角作为增强描述的开头（例如 "A static roadside camera captures..."、"A fixed overhead camera records..."）。
> 4. 自然地纳入给定的天气和周边上下文，但不要改变原始环境。
> 5. 如果添加对象，请将多个实例放置在不同且真实的位置或上下文中。
> 6. 仅将车辆放置在道路上；绝不要放在人行道上。
> 7. 不要将自行车放在车架上。
> 8. 确保所有对象位置都符合现实世界的物理规律和光照条件。
> 9. 从描述中移除所有绿植、树木和灌木。
> 10. 光照必须是与给定天气一致的自然日光。
> 11. 避免使用不真实或过于戏剧化的光照描述，例如 "sun reflection"、"glaring sunlight"、"blinding light" 或不自然的镜头效果。仅描述现实生活中会出现的光影效果。
> 12. 避免使用 serene、mystical ambience 等描述。
> 13. 避免提供不必要的天气或周边环境描述。
> 14. 不要在道路上添加废弃、抛锚或受损车辆。
> 15. 不要在道路上添加没有驾驶员或可见乘员的车辆。
>
> 特定上下文增强规则：
>
> 1. 当周边环境表明存在特定文化或地区背景时（例如 "Indian traffic scene"、"European city"、"Southeast Asian street"），请融入合适的上下文元素：
>     - 对于 Indian 背景：包含 auto-rickshaw、多样化车辆类型（cars、motorcycles、scooters、buses）、混合交通模式、在适合时加入街头小贩，以及多样化着装风格
>     - 对于 European 背景：包含典型的 European 车辆、有序的交通模式，以及恰当的建筑风格参考
>     - 对于 Asian 背景：包含地区特有车辆（scooters、small cars、delivery bikes）以及合适的城市密度
> 2. 如果未指定周边环境、为空或为 "none"，则使用不带特定文化标记的通用交通元素。
> 3. 在保持真实感的同时，调整车辆类型和交通模式，使其与指定的周边环境相匹配。
> 4. 加入与周边环境相符、具有文化合理性的人类元素（服装、行为、人口特征）；若未指定周边环境，则使用通用元素。
> 5. 添加人物时，确保其外观、穿着和活动与指定周边环境在上下文上相符；若未指定周边环境，则使用中性描述。
> 6. 对于交通场景，应体现指定地区或文化中典型的交通构成与行为模式；如果未给定具体上下文，则使用标准混合交通。
>
> 输出格式：
>
> - 以单段文本返回增强后的描述，并用 <...> 包裹。
> - 不要包含任何额外文本或解释。
>
> 输入：
>
> - Camera angle: {camera_angle}
> - Weather: {weather}
> - Surroundings: {surroundings}
> - Original Caption: {caption}"""

在这个示例中，我们使用了不同的相机角度（topdown、front dashboard）、不同的天气（snow、fog、night），以及 surroundings = none。

### Cosmos Predict 2 输出图像

下方示例是一个由多次生成结果组成的网格，涵盖了不同的相机角度和天气条件。

![输出图像](assets/output.jpg)

## 训练下游 ITS 检测器

为了说明 Cosmos Predict 2 图像生成对下游 ITS 检测器的影响，我们使用 Cosmos Predict 2 生成的图像训练了一个 RT-DETR 检测器；这些图像覆盖不同的相机角度、天气和目标对象。随后，我们在三个公开 KPI 上对训练后的模型进行了评估。

## 结果

实验使用了约 ~62k 张真实 ITS 图像和约 ~165K 张由 Cosmos Predict 2 生成的合成图像（未做任何筛选），并采用了本教程中介绍的流水线。

作为对比，我们使用一个基线 ITS 检测器，其采用完全相同的架构，并仅在约 ~220k 张真实图像上训练（不含合成数据）。两个模型都使用相同的骨干网络：在 OpenImages 上预训练的 ResNet‑50。

以下是三个 KPI 上的结果：

## ACDC 数据集

ACDC 数据集是一个与 Intelligent Transportation System (ITS) 相关的数据集，包含 snow、fog、rain 和 night 等不同天气条件。该数据集约有 ~2k 张图像，可在此处获取：<https://acdc.vision.ee.ethz.ch/>

以下是在所有天气条件下，数据集中最常见对象（car、person、bicycle）的 AP50 结果：
![ACDC 结果图](assets/acdc_plots.png)
如上所示，蓝色曲线（使用 Cosmos Predict 2 生成图像训练得到的检测器）在所有天气/光照条件和各类对象上都持续获得高于红色曲线的 AP50。

## SUTD 数据集

SUTD 数据集也是一个 ITS 相关数据集，天气条件更加多样，包括 snow、fog、rain、night、cloudy 和 sunny。该数据集约有 ~10k 张图像，可在此处获取：<https://sutdcv.github.io/SUTD-TrafficQA/#/download>

以下是在所有天气条件下，数据集中最常见对象（car、person、bicycle）的 AP50 结果：
![SUTD 结果图](assets/sutd_plots.png)
如上所示，蓝色曲线（使用 Cosmos Predict 2 生成图像训练得到的检测器）在所有天气/光照条件和各类对象上都持续获得高于红色曲线的 AP50。

## DAWN 数据集

DAWN 数据集也是另一个 ITS 相关数据集，包含 snow、fog、rain 和 sandy 等不同天气条件。该数据集约有 ~1k 张图像，可在此处获取：<https://www.kaggle.com/datasets/shuvoalok/dawn-dataset>

以下是在所有天气条件下，数据集中最常见对象（car、person）的 AP50 结果：
![DAWN 结果图](assets/dawn_plots.png)
如上所示，蓝色曲线（使用 Cosmos Predict 2 生成图像训练得到的检测器）在所有天气/光照条件和各类对象上都持续获得高于红色曲线的 AP50。

## 结论

本教程展示了 Cosmos Predict 2 如何通过策略性生成 ITS 图像来应对长尾稀疏、视角/天气多样性和领域差距问题，从而为下游检测器带来可量化的性能提升。

在我们的实验中，基线模型使用约 ~220k 张真实图像，而 Cosmos 方案使用约 ~62k 张真实图像和 ~165k 张合成图像。Cosmos 方案在 bicycle 类别上表现出显著提升，其 AP50 改进值是按每个 KPI 在不同天气条件下取平均得到的。

测得的 bicycle AP50 提升（baseline → +Cosmos Predict 2）：

- ACDC: 0.287 → 0.374 (+0.087)
- SUTD: 0.515 → 0.610 (+0.095)
- DAWN: 0.514 → 0.801 (+0.286)

关键结论：

1. 有针对性的 text-to-image 合成能够在保持视觉真实感的同时，提高低频类别（例如 bicycle）的准确率。
2. 对视角、光照和天气进行系统遍历，有助于提升模型在真实部署条件下的泛化能力。
3. 强有力的提示词设计（captioning + 受约束增强）是获得可控且高质量输出的关键。

有关实现和训练配置的更多细节，请参阅本仓库附带的设置和配置文件。

---

## 文档信息

**发布日期：** 2025 年 11 月 18 日

### 引用

如果你使用了本配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_cosmos_predict_2_2025,
  title={Cosmos Predict 2 Text2Image for Intelligent Transportation System (ITS) Images},
  author={Verma, Charul and Entezari, Reihaneh and Jain, Arihant and Devendran, Dharshi and Kumar, Ratnesh},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/predict2/inference-its/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Charul Verma、Reihaneh Entezari、Arihant Jain、Dharshi Devendran 与 Ratnesh Kumar（2025）。Cosmos Predict 2 Text2Image for Intelligent Transportation System (ITS) Images。收录于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/predict2/inference-its/inference.html>
