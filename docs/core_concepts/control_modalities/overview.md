# Cosmos Transfer 2.5 控制模态：核心概念

> **作者：** [Aiden Chang](https://www.linkedin.com/in/aiden-chang/) • [Akul Santhosh](https://www.linkedin.com/in/akulsanthosh/)
> **机构：** NVIDIA

## 概览：控制的挑战

本文档是一份关于使用 **Transfer 2.5** 视频生成模型所需 **核心概念** 的综合指南。要想获得理想效果，关键在于理解 **Guidance Scale**（你的文本提示词）与四种主要 **控制模态**——Edge、Depth、Segmentation 和 Vis——之间的平衡关系。

**核心结论：** 要获得高保真、结构一致的视频结果，必须进行多控制调优——也就是有策略地组合使用 Edge、Vis、Depth 和 Seg 等模态。

---

## 1. 关键概念：控制强度

### 1.1. Guidance Scale（提示词强度）

这一原则决定了模型会在多大程度上严格遵循你的文本提示词，而不是视觉控制信号。

- **作用**：控制文本提示词的影响力。
- **良好起点**：Guidance = 3
- **何时提高**：如果视觉输出没有体现提示词中描述的改动（例如试图把一件衬衫改成某种特定材质），请提高到 **5+**。

### 1.2. 控制权重归一化（非常重要）

这一原则决定了模型如何平衡多个控制模态（例如 Edge + Seg + Vis）之间的影响力。

- **规则 1：如果所有控制权重的总和小于或等于 `1.0`，则不会归一化。** 权重会按原值直接使用。
  - *示例：* `{seg: 0.2, edge: 0.2}`（总和为 0.4）会按原值使用。
- **规则 2：如果总和大于 `1.0`，则会进行归一化。** 权重会按比例重新缩放，使新的总和等于 `1.0`。
  - *示例：* `{seg: 4.0, edge: 1.0}`（总和为 5.0）会被归一化，并按 `{seg: 0.8, edge: 0.2}` 运行。

---

## 2. 技术细节：控制模态

系统使用四种主要模态，为视频注入结构一致性、语义一致性、相对关系一致性和视觉一致性。

![整体架构](assets/Cosmos-Transfer2-2B-Arch.png)

### 2.1. Edge 控制（结构保持）

- **功能：** 保留视频的 **原始结构、形状和布局**。
- **最适合：** 更改纹理、服装或光照，同时需要保持底层形状不变的场景。
- **局限性：** 当尝试大幅改变对象形状时效果较差（例如把一件衬衫变成香蕉）。

Edge control 在 Cosmos Transfer 2.5 [代码仓库](https://github.com/nvidia-cosmos/cosmos-transfer2.5)中得到原生支持。用户也可以选择自行提供边缘检测输出，只需提供一个指向预计算边缘控制视频的 `control_path`。如果没有提供 `control_path`，Cosmos Transfer 2.5 会自动动态生成 edge control 模态。

当目标物体与背景的轮廓过于相似时，默认的 Canny 边缘检测可能无法稳定地区分它们。在这种情况下，先对视频做亮度和对比度调整，有助于在送入 Cosmos Transfer 2.5 流水线前生成更干净、更稳定的边缘图。

下面给出一个预处理实现示例：

```python
import cv2, os

def generate_edges(in_path, out_path):
    cap = cv2.VideoCapture(in_path)
    assert cap.isOpened(), "Could not open input video."
    fps = cap.get(cv2.CAP_PROP_FPS)
    w = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
    h = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))

    bright = 50
    contrast = 1.0

    fourcc = cv2.VideoWriter_fourcc(*"mp4v")  # use "avc1" if you prefer H.264 and it's available
    out = cv2.VideoWriter(out_path, fourcc, fps, (w, h), isColor=False)

    while True:
        ok, frame = cap.read()
        if not ok:
            break

        frame = cv2.convertScaleAbs(frame, alpha=contrast, beta=bright)
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        blurred = cv2.GaussianBlur(gray, (3, 3), 1.4)
        edges = cv2.Canny(blurred, 10, 50)
        out.write(edges)
    cap.release()
    out.release()

if __name__ == "__main__":
    in_path = "input_video.mp4"
    out_path = "edges.mp4"
    generate_edges(in_path, out_path)
```

### 2.2. Segmentation（Seg）控制（结构变化与语义替换）

- **功能：** 支持 **大幅结构变化** 与语义替换，可用于彻底变换或替换物体、人物或背景。
- **最适合：** 当提示词要求大变化时，生成逼真的 *新* 物体或场景。
- **局限性：** 权重过高可能导致 **“幻觉”**（不真实或物理上不正确的对象）。
- **推荐用法：** 使用 Seg 时，**始终** 配合你想改动区域的 **mask**，并且 **始终** 将其作为 **多控制** 配置的一部分使用（例如与 Edge 搭配）。

生成 segmentation mask 有两种方式：

1. **手动指定对象：** 你可以提供想要分割的对象列表，并在 Cosmos Transfer 2.5 仓库中运行 SAM2 endpoint。
实现可见于 [SAM2 pipeline code](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/cosmos_transfer2/_src/transfer2/auxiliary/sam2/sam2_pipeline.py)。
2. **自动目标检测（推荐用于大规模处理）：** 对于更大的数据集，你可以使用 [RAM++](https://github.com/xinyu1205/recognize-anything) 之类的模型自动检测对象。检测得到的对象标签随后会传入 Cosmos Transfer 2.5 流水线，并使用与上文相同的 SAM2 工作流生成 segmentation mask。

### 2.3. Vis 控制（光照与背景氛围）

- **功能：** 保留原始视频的 **背景、光照和整体外观**。默认情况下，它会施加轻微的平滑/模糊效果，但底层视觉特征保持不变。
- **最适合：** 作为 Edge 或 Seg 的 *补充*，通常配合 **较低权重** 使用，以微调视觉一致性。
- **直觉理解：**
  - **提高 Vis 权重** 可以保留更多原始视频的外观。
  - **降低 Vis 权重** 则允许相对于原视频进行更多改动（但权重过低可能会 *增加* 背景幻觉）。
- **局限性：** 如果权重过高，输出将基本退化为原始视频。已知对 Vis control 进行 mask 会引发幻觉。

Vis control 在 Cosmos Transfer 2.5 [代码仓库](https://github.com/nvidia-cosmos/cosmos-transfer2.5)中原生内置，无需指定 `control_path`。

### 2.4. Depth 控制

- **功能：** 通过遵循距离与透视关系来维持 **3D 真实感** 和 **空间一致性**。
- **潜在用途：** 在向场景中放置新物体或保持相机运动连贯性时会有帮助。
<!-- - *文档撰写中。* -->

### 2.5 示例

下图展示了由原始视频生成的不同控制模态：

| **控制类型** | **说明** | **示例** |
|-----------------|-----------------|-------------|
| **原始视频** | 源视频输入 | <video src="assets/wave.mp4" controls width="300"></video> |
| **Edge** | 物体与基础设施的几何边界 | <video src="assets/edge.mp4" controls width="300"></video> |
| **Segmentation** | 场景的语义分割 | <video src="assets/seg.mp4" controls width="300"></video> |
| **Vis** | 保留背景与光照的模糊表示 | <video src="assets/vis.mp4" controls width="300"></video> |

---

## 2.5. 二值 Mask：局部化控制

Masking 是一种将控制模态应用到视频帧特定区域的技术。

- **机制：** 使用二值 mask（黑白图像/视频）。控制模态 **只会应用到 mask 中的白色像素**。
  - **白色像素：** **变化/施加控制** 的区域。
  - **黑色像素：** 应当 **保持不变** 或需要抑制控制的区域。
- **Seg Masking（标准做法）：** 这是 masking 的一种 **有效** 且标准的使用方式。你向 Seg control 输入一个 mask，告诉它具体应在何处执行语义替换。
- **Vis Masking（避免使用）：** 已知对 Vis control 做 masking 会导致视觉 **幻觉**，通常不推荐。应以较低权重全局使用 Vis。

<video width="720" controls>
  <source src="assets/mask.mp4" type="video/mp4">
  您的浏览器不支持 video 标签。
</video>

---

## 3. 建立直觉：多控制调优

为了实现背景替换等复杂目标，你必须有策略地组合多种模态。

| 如果你的目标是…… | 提高这个设置 | 降低这个设置 |
| :---- | :---- | :---- |
| **减少幻觉 / 奇怪物体** | | Seg 权重或 Guidance |
| **保留原始背景/光照** | Vis 权重 | |
| **保持原始视频结构** | Edge 权重 | |
| **实现更逼真的大幅变化** | Seg 权重 | Vis 权重 |
| **保持物体边界一致** | Edge 权重 | Vis 权重 |

### 背景替换演练

下面的步骤展示了如何在一个基础视频上逐步叠加各类控制，从而实现高保真结果。目标是把原始视频的背景替换成户外街道环境。这并不是一套执行背景替换的固定配方；相反，它是一个帮助你理解每种控制模态如何影响结果的演练。有关如何生成这些结果的具体指南，请参见[这里](../../recipes/inference/transfer2_5/inference-real-augmentation/inference.md)。

#### 第 1 步：仅使用 Edge（基础结构）

第一步通常是应用 **Edge control**，以保持核心结构（例如人物的动作姿态）。

- **操作：** 使用 **过滤后的 edge map**（仅保留人物的边缘）应用 Edge control。
- **结果直觉：** 人物运动得以保留，但背景依然显得不真实且有扭曲。这表明 Edge 只能控制形状，**不能保证视觉保真度**。

<video width="500" controls>
<strong>Mask 视频</strong>
  <source src="assets/only_edge.mp4" type="video/mp4">
  您的浏览器不支持 video 标签。
</video>

#### 第 2 步：Edge + Vis（增加光照一致性）

为了修复不真实的观感并保留相机效应，需要加入 **Vis control**。

- **操作：** 加入 **Vis control**，并设置中等权重（例如 0.6）。
- **结果直觉：** 鱼眼畸变更准确，背景也更不模糊。这印证了 **Vis** 在保留整体 **视觉氛围** 与相机属性方面的作用。不过整体真实感仍然不足。

<video width="500" controls>
<strong>Mask 视频</strong>
  <source src="assets/edge_with_vis.mp4" type="video/mp4">
  您的浏览器不支持 video 标签。
</video>

#### 第 3 步：Edge + Vis + Seg（注入真实感）

为了生成一个 *全新且逼真* 的背景，需要使用 **Segmentation control**。

- **操作：** 加入 **Seg control**，设置中等权重（例如 0.4），并使用 **mask**（背景区域为白色）只对背景区域执行语义替换。
- **结果直觉：** 最终输出在视觉上清晰且一致。**Seg** 提供了生成合理新环境所必需的 **语义信息**，而 **Edge** 与 **Vis** 则确保主体和光照保持一致。

<video width="500" controls>
<strong>Mask 视频</strong>
  <source src="assets/street_background.mp4" type="video/mp4">
  您的浏览器不支持 video 标签。
</video>

完整 recipe 可见：[使用 Cosmos Transfer 2.5 进行真实世界视频编辑指南](../../recipes/inference/transfer2_5/inference-real-augmentation/inference.md)。

---

## 最佳实践

**建议这样做：**

- ✅ **使用多控制**：对复杂任务组合使用多种模态（尤其是 Seg + Edge）。
- ✅ **在 Seg 中使用 Mask**：使用 Seg control 时提供 mask，以隔离需要修改的区域。
- ✅ **Vis 从较低权重开始**：将 Vis 作为补充，以较低权重（例如 0.4-0.6）来维持视觉氛围。

**不要这样做：**

- ❌ **单独使用 Seg Control**：不要只使用 Segmentation control；这会导致非常不真实的结果。
- ❌ **使用 Seg + Vis（不含 Edge）**：不推荐这种组合，因为它可能导致不可预测的结果。
- ❌ **对 Vis Control 做 Mask**：避免对 Vis control 做 masking，因为已知会引发幻觉。
- ❌ **给 Vis 设置过高权重**：非常高的 Vis 权重只会返回你的原始视频——应使用[较低权重]。

## 用例

- [使用 Cosmos Transfer 2.5 进行真实世界视频编辑指南](../../recipes/inference/transfer2_5/inference-real-augmentation/inference.md)。

---

## 文档信息

**发布日期：** 2025 年 11 月 9 日

### 引用

如果你使用了本内容或引用了本工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_control_modalities_2025,
  title={Cosmos Transfer 2.5 Control Modalities: Core Concepts},
  author={Chang, Aiden and Santhosh, Akul},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/control_modalities/overview.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用格式：**

> Aiden Chang 与 Akul Santhosh（2025）。Cosmos Transfer 2.5 控制模态：核心概念。收录于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/control_modalities/overview.html>

