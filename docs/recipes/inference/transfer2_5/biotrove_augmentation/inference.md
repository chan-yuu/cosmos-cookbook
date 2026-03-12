# 使用 Cosmos Transfer 2.5 对 BioTrove 飞蛾进行域迁移

> **作者：** [Paula Ramos, PhD](https://www.linkedin.com/in/paula-ramos-phd/)
> **机构：** [Voxel51](https://voxel51.com/)

## 概述

本配方展示了一个完整的**域迁移流水线**，用于借助 **Cosmos Transfer 2.5** 和 [**FiftyOne**](https://docs.voxel51.com/) 解决 BioTrove moth dataset 中的**数据稀缺**问题。
即使缺少控制信号（深度、分割），它也展示了如何通过 **基于边缘的控制**、**纯 Python 推理** 和 **FiftyOne 可视化**，将静态图像转换为逼真的农业场景。

> 本配方使用 [FiftyOne](https://docs.voxel51.com/)，这是 [Voxel51](https://voxel51.com/) 的开源工具包，用于可视化、清洗和评估计算机视觉数据集。Voxel51 构建的工具可帮助研究人员和工程师更好地理解其数据并提升模型性能。

> 请访问 [FiftyOne 教程](https://docs.voxel51.com/tutorials/cosmos-transfer-integration.html) 一次性运行完整流程。

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| [Cosmos Transfer 2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5) | 推理 | 用于稀缺生物数据集的域迁移 |

---

## 设置

在运行此配方前，请先完成环境配置：
**[设置与系统要求](setup.md)**

---

## 动机：BioTrove 飞蛾中的数据稀缺与域差距

[**BioTrove**](https://baskargroup.github.io/BioTrove/) 数据集是一个大规模多模态集合，但其中存在明显的**类别不平衡**。飞蛾属于**样本最少的类别之一**，并且大多数样本采集于**实验室或人工室内背景**，而不是农业环境。这带来了两个重要挑战：

- **数据稀缺** —— 自然田间条件下的飞蛾真实场景很少
- **域差距** —— 在实验室风格图像上训练的模型难以泛化到户外农业环境

为了构建稳健的分类器，我们使用了 [**FiftyOne 的 BioCLIP 语义搜索**](https://github.com/paularamo/fiftyone-workshop-biodiversity) 从完整数据集中检索出 **约 1000 张飞蛾图像**。然而，大多数检索结果仍然缺少逼真的田间背景。该子数据集位于 Hugging Face Hub，可在[这里](https://huggingface.co/datasets/pjramg/moth_biotrove)找到。

Cosmos Transfer 2.5 让我们能够在保留每只飞蛾结构与身份的同时，**将这些稀缺的实验室风格图像转换为照片级真实的农业场景**。内部实验表明，借助更好的域对齐和更丰富的外观多样性，分类准确率可提升 **20–40%**。

本配方演示了一个完整且可复现的流水线，它可以：

- 将飞蛾图像转换为视频
- 生成基于边缘的控制视频
- 运行 Cosmos Transfer 2.5 推理
- 在 FiftyOne 中构建多模态分组数据集
- 计算嵌入 + 相似性搜索
- 大规模生成逼真的农业飞蛾场景

---

## 流水线概览

以下是端到端流程：

1. 使用 BioCLIP 语义搜索**筛选并检索飞蛾图像**。已提供子数据集。
2. **将图像转换为视频**（Cosmos 需要视频输入）
3. **生成 Canny 边缘图**作为控制信号
4. **创建 JSON spec files**，供 Cosmos Transfer 2.5 使用
5. **运行 Cosmos-Transfer 推理**（纯 Python 调用）
6. **从生成视频中提取最后一帧**
7. **在 FiftyOne 中构建分组数据集**，实现同步切片
8. **计算嵌入 + 相似性搜索**
9. **可视化结果**（并排对比、嵌入、UMAP）

---

### 1. 使用 BioCLIP 提取具有代表性的子数据集

我们通过以下方式筛选 BioTrove：

- **语义搜索**
- **文本查询（"moth"）**
- **使用 BioCLIP embeddings 的向量相似性**

使用：

```python
import fiftyone as fo
import fiftyone.utils.huggingface as fouh

dataset_src = fouh.load_from_hub(
    "pjramg/moth_biotrove",
    persistent=True,
    overwrite=True,
    max_samples=1000,
)
```

> ![file_name](https://cdn.voxel51.com/tutorials/cosmos-transfer2_5/moth_biotrove.webp)

---

### 2. 准备输入：将图像转换为视频

Cosmos Transfer 2.5 当前支持将**视频**作为推理输入。
我们通过 FFmpeg 将每张图像转换为 **10-frame MP4 clip**。

Python 版本：

```python
import os, subprocess
from pathlib import Path

videos_root = images_root.parent / "videos"
videos_root.mkdir(exist_ok=True)

for img in sorted(images_root.glob("*.jpg")):
    output = videos_root / f"{img.stem}.mp4"
    cmd = [
        "ffmpeg", "-y", "-loop", "1",
        "-i", str(img), "-t", "1",
        "-vf", "pad=ceil(iw/2)*2:ceil(ih/2)*2",
        "-c:v", "libx264", "-pix_fmt", "yuv420p",
        str(output),
    ]
    subprocess.run(cmd, check=True)
```

---

### 3. 生成边缘图（控制信号）

由于 BioTrove 图像缺少深度/分割信息，我们创建 **Canny 边缘视频** 作为控制：

> ![file_name](https://cdn.voxel51.com/tutorials/cosmos-transfer2_5/edge_control.webp)

这样可以在允许风格变换的同时确保结构得以保留。

```python
import cv2

def make_edge_video(input_video, output_video):
    cap = cv2.VideoCapture(str(input_video))
    fps = cap.get(cv2.CAP_PROP_FPS) or 24
    w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
    h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))

    out = cv2.VideoWriter(str(output_video),
                          cv2.VideoWriter_fourcc(*"mp4v"),
                          fps, (w,h), isColor=False)

    while True:
        ret, frame = cap.read()
        if not ret:
            break
        edges = cv2.Canny(cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY), 80, 180)
        out.write(edges)

    cap.release(); out.release()
```

---

### 4. 为 Cosmos-Transfer 构建 JSON spec files

每个视频都需要一个 JSON spec，用于描述：

- prompt
- negative prompt
- guidance
- control path
- resolution
- steps

```python
def write_spec_json(spec_path, video_abs, edge_abs, name):
    obj = {
        "name": name,
        "prompt": MOTH_PROMPT,
        "negative_prompt": NEG_PROMPT,
        "video_path": str(video_abs),
        "guidance": GUIDANCE,
        "resolution": RESOLUTION,
        "num_steps": NUM_STEPS,
        "edge": {
            "control_weight": 1.0,
            "control_path": str(edge_abs),
        },
    }
    spec_path.write_text(json.dumps(obj, indent=2))
```

在这一步中，我们为每个输入视频构建一个 JSON spec。
每个 spec 控制以下内容：

```prompt``` / ```negative_prompt``` —— 描述目标域的文本提示（通常由 LLM 生成或扩展），例如户外田野影像、自然光照、真实植被和飞蛾外观。我们正是在这里引入光照、背景/环境以及飞蛾纹理/外观的变化，同时保持生物语义不变。

```video_path``` —— 原始域视频的路径。

```edge.control_path``` + ```edge.control_weight``` —— Canny 边缘控制视频的路径及其权重，用于约束结构和运动。

```guidance```、```resolution```、```num_steps``` —— Cosmos Transfer 2.5 使用的生成超参数。

这些提示（```MOTH_PROMPT``` 及其变体）先统一编写，再由 LLM 扩展为多个逼真的变体（光照、环境、纹理），随后嵌入到针对同一基础视频的不同 JSON spec files 中。

> 如需查看该配置，请参阅 [FiftyOne 教程](https://docs.voxel51.com/tutorials/cosmos-transfer-integration.html)

---

### 5. 运行 Cosmos Transfer 2.5 推理

纯 Python 调用：

```python
cmd = [sys.executable, str(INFER_SCRIPT), "-i", str(spec_json), "-o", str(OUT_DIR)]
subprocess.run(cmd, check=True)
```

运行命令后，请注意以下参数定义：

- ```INFER_SCRIPT``` —— 你要执行的 Cosmos Transfer 2.5 推理脚本路径。
- ```SPEC_JSON``` —— 定义模型、输入和控制信号的 JSON specification file 路径。
- ```OUT_DIR``` —— 用于保存生成视频、日志和元数据的输出目录。

---

### 6. 提取最后一帧

```python
last_png = extract_last_frame(out_vid, last_frames_dir)
```

> ![file_name](https://cdn.voxel51.com/tutorials/cosmos-transfer2_5/last_frame.webp)

---

### 7. 在 FiftyOne 中构建分组数据集

在 FiftyOne 中，grouped dataset 是包含多个 slice 的数据集。在 grouped dataset 的上下文中，slice 指的是每个组中的一个组成部分（例如图像、视频或点云）。每个组可以包含多个 slice，且这些 slice 可能具有不同模态，并统一组织在 group field 下。[Grouped Datasets](https://docs.voxel51.com/user_guide/groups.html)

生成的切片包括：

- `image`
- `video`
- `edge`
- `output`
- `output_last`

> ![file_name](https://cdn.voxel51.com/tutorials/cosmos-transfer2_5/grouped_dataset.webp)

这些切片支持在应用中进行同步的并排比较。

---

### 8. Embeddings 和相似性搜索（CLIP）

该工作流使用 FiftyOne Model Zoo 中的 ```CLIP``` 模型，为数据集视图（```flattened_view```）中的每个样本生成 embeddings。生成的 embeddings 存储在 ```embeddings``` 字段中。随后，系统使用这些 embeddings 创建 similarity index，从而支持你在数据集中执行相似性搜索——例如查找视觉上或语义上相似的样本。brain_key ```key_sim``` 用于在后续查询中引用该 similarity index。

```python
model = foz.load_zoo_model("clip-vit-base32-torch")
flattened_view.compute_embeddings(model, embeddings_field="embeddings")

fob.compute_similarity(
    flattened_view,
    model="clip-vit-base32-torch",
    embeddings="embeddings",
    brain_key="key_sim",
)
```

> ![file_name](https://cdn.voxel51.com/tutorials/cosmos-transfer2_5/embeddings.webp)

---

### 9. 结果与观察

虽然 Cosmos Transfer 2.5 生成了较高比例的可用样本，但整体可用性仍取决于控制信号的优化。尤其是，改进 edge-control 的生成会带来更稳定的几何结构和更少的最终输出伪影。一个很有前景的下一步是引入诸如 SAM3 之类的 semantic segmentation model 来生成干净的飞蛾掩码。这样可以更好地保留昆虫形态，并减少域迁移阶段中昆虫形状发生变化的情况。

即使有这些控制手段，也不是每个合成样本都适合用于训练。每个输出仍应经过质量检查步骤。

- 输出是逼真的农业场景
- 飞蛾形态得到保留*
- 背景多样性增加
- Edge 控制主要保持飞蛾结构——这一点仍需继续改进
- 视觉连贯性高

> morphology 指的是飞蛾实际的物理结构，包括其形状、翅膀轮廓、触角、身体比例和整体几何形态。
>
---

## 结论

本配方展示了：

- 如何应对 **dataset scarcity**
- 如何创建逼真的域迁移增强数据
- 如何将 FiftyOne 与 Cosmos-Transfer 集成
- 如何构建可复现的物理 AI 数据流水线

这种方法可推广到：

- 其他昆虫/动物数据集
- 医疗数据稀缺场景
- 机器人感知中的域差距问题
- 任何缺少真实世界多样性的场景

关于环境设置，请参阅[设置指南](setup.md)。当你的环境准备就绪后，请使用这个[教程](https://docs.voxel51.com/tutorials/cosmos-transfer-integration.html)一次性运行全部流程。
如需更多示例，你可以探索 cookbook 中其他的 Cosmos-Transfer 配方。

---

## 文档信息

**发布日期：** 2025 年 11 月 26 日

### 引用

如果你使用了此配方或引用了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_domain_transfer_for_2025,
  title={Domain Transfer for BioTrove Moths with Cosmos Transfer 2.5},
  author={Ramos, Paula},
  organization={Voxel51},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/biotrove_augmentation/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Paula Ramos（2025）。使用 Cosmos Transfer 2.5 对 BioTrove 飞蛾进行域迁移。载于 *NVIDIA Cosmos Cookbook*。Voxel51。可访问：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/transfer2_5/biotrove_augmentation/inference.html>
