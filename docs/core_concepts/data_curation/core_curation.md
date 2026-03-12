# 使用 Cosmos-Curator 进行核心数据整理

> **作者：** [Jibin Varghese
 ](https://www.linkedin.com/in/jibinrajan/) • [Amol Fasale](https://www.linkedin.com/in/amolfasale/) • [Jingyi Jin](https://www.linkedin.com/in/jingyi-jin/)
>
> **机构：** [NVIDIA](https://www.nvidia.com/)

**前置要求：** 在继续进行核心数据整理之前，请确保你已经完成了 [数据整理概览](overview.md) 中列出的前置步骤，包括数据获取、采样和可视化。

现在，你的数据集已经完成归一化和采样，可以更好地理解其特征。接下来的步骤包括：将视频切分为更短的片段、为视频片段生成字幕、应用过滤，并在需要时将数据切分为 webdataset 格式。这些核心整理任务都可以通过 [Cosmos-Curator](https://github.com/nvidia-cosmos/cosmos-curate) 完成。

Cosmos Curator 工具提供了多种部署模式，以适配不同的使用场景和技术需求：

## 部署选项

1. **NVCF（预部署函数）** — *推荐新手使用*

    为了简化整理流程，Cosmos Engineering 团队已在 8x H100 GPU 基础设施上部署了一个预配置的 NVCF 函数。当你处理的原始数据相对干净，不需要大量过滤或自定义分类时，这一选项非常合适。它配置简单、上手快速。

2. **本地基础设施** — *使用 Docker 获得完全控制*

    若你希望完全掌控整理流水线，可以使用 Docker 容器在本地运行 Cosmos-Curator。这样你可以创建自定义过滤器、修改源代码，并在自己的硬件上运行任务。此选项非常适合开发、测试以及较小规模的数据集。

    > **注意**：如果你的本地系统只有单块 GPU，Cosmos-Curator 会自动禁用 `STREAMING` 模式，因为它无法同时维持多个 GPU 阶段；在这种情况下，如果输入视频的尺寸/体量较大，Ray worker 可能会遇到内存不足（OOM）问题。

3. **SLURM 集群** — *面向大规模的高性能计算*

    对于大规模处理任务，你可以在配备多块 GPU 的 SLURM 集群上部署 Cosmos-Curator。该选项可提供最大的计算能力，适合在生产环境中处理海量数据集。

## 使用预部署的 NVCF 函数

NVCF 方案提供了一种精简的云端数据整理方式，无需本地基础设施。

![](images/nvcf.png)

### 前置要求

安装 Cosmos Curator CLI 客户端（详细安装说明见下文的[本地运行部分](#running-cosmos-curator-locally)）：

```shell
# Quick setup (full instructions in Local Setup section)
git clone --recurse-submodules https://github.com/nvidia-cosmos/cosmos-curate.git
cd cosmos-curate
uv venv --python=3.10 && source .venv/bin/activate
uv pip install poetry && poetry install --extras=local
```

### NVCF 配置

> ⚠️ **安全警告：** 请将 API key 存储在环境变量或安全密钥管理系统中（例如 HashiCorp Vault、AWS Secrets Manager）。绝不要将 API key 提交到源代码仓库，也不要以明文形式分享。

1. **设置 NGC API key**：如果你还没有有效的 API key，可按以下方式获取：访问 <https://org.ngc.nvidia.com/setup>，在左上角选择 **Cosmos API**，并将组织配置文件选择为 **no team**。点击 **Generate API Key**，然后选择 **Generate Personal Key**。生成后请妥善保存该 key 以备后用（该 key 只会显示一次）。

    ```shell
    cosmos-curate nvcf config set --org <NVCF Org ID> --key <NGC NVCF API Key>
    # in case you will upload helm chart to you org
    ngc config set
    ```

2. **注册活动函数**，即 Cosmos Engineering 团队预先部署好的函数：

    ```shell
    cosmos-curate nvcf function import-function \
    --funcid 58891eb4-0eef-4a91-8177-5c272275a375 \
    --version b109565a-e15e-492c-997d-3c35ed2c7a9e \
    --name Cosmos-Curate-LHA-1_0_2
    ```

### 使用 NVCF 进行视频切分与字幕生成

处理数据集时，建议采用如下目录结构：

```
project_root/
└── dataset_root/
    ├── raw_data/          # Raw data
    ├── processed_data/    # Processed data
        ├── v0/            # Version 0
            ├── clips/     # Processed clips
            ├── meta/      # Metadata
            ├── embeds/    # Calculated embeddings
            └── ...
        └── v1/            # Version 1
            └── ...
    └── webdataset/        # Curated and sharded webdataset
```

之后，你就可以调用该函数，对你的 S3 bucket 执行数据整理。后续步骤——例如切分、字幕生成、重新字幕生成和分片——都使用相同的命令结构；你只需要修改 JSON 配置文件，以指定云函数应执行的具体功能。

```shell
cosmos-curate nvcf function invoke-function --data-file <path_to_your_json_file> --s3-config-file <path_to_file_containing_a_single_aws_cred>
```

下面给出一个面向 Intelligent Transportation Systems（ITS）数据集初始处理的 JSON 示例。请根据你的具体场景微调这些参数。

```json
{
    "pipeline": "split",
    "args": {
        "input_video_path": "s3://your_bucket/your_dataset/raw_data",
        "output_clip_path": "s3://your_bucket/your_dataset/processed_data/v0",
        "generate_embeddings": true,
        "generate_previews": true,
        "generate_captions": true,
        "splitting_algorithm": "transnetv2",
        "captioning_algorithm": "qwen",
        "captioning_prompt_variant": "default",
        "captioning_prompt_text": "You are a video caption expert. Please describe this video in two paragraphs: 1. Describe the static scene: What objects and people are present? What are their spatial relationships and the overall environment? 2. Describe the actions and motion: How do objects move and interact? Focus on object permanence, collisions, and physical interactions. Be specific and precise in your observations.",
        "limit": 0,
        "limit_clips": 0
   }
}
```

启用 `generate_embeddings`、`generate_previews` 和 `generate_captions` 参数后，会生成对应的特征。`splitting_algorithm` 参数支持两种选项：`transnetv2`（默认）和 `fixed-stride`。`caption_algorithm` 参数定义用于视频字幕生成的模型（默认是 `Qwen2.5-7b-VLM-instruct`），而 `captioning_prompt_text` 参数则提供系统/用户指令，用于引导字幕生成。在该配置中，字幕模型会先识别交通监控片段中的静态场景元素，再描述动态行为或物体之间的空间关系。

为了提升生成字幕的质量和针对性，可以在相应的 JSON 项中配置更复杂的提示词。一个视频片段可利用的上下文信息越丰富，就越能更有效地嵌入提示词中，从而提升最终字幕的相关性与准确性。

### 字幕生成最佳实践

在为字幕模型设计提示词时，请遵循以下原则：

- **推理-训练一致性**：让字幕结构与推理阶段的提示词格式保持一致。例如，先描述首帧，再描述动作序列。这种对齐有助于模型在推理时学会响应类似的提示结构。

- **相机视角区分**：在字幕开头显式写明相机视角（例如 CCTV、dashcam、bodycam），帮助模型区分不同视觉视角，并生成与视角一致的输出。

- **噪声抑制**：在线视频来源通常包含 logo、水印和文字叠加等视觉噪声。提示词应引导字幕模型聚焦于相关场景元素并忽略这些伪影，以防模型在训练时受到干扰。

- **以运动为中心的描述**：在字幕中优先描述动态元素和物理交互，而不是静态描述。这有助于模型学习对生成任务至关重要的运动模式和物理真实感。

- **领域特定术语**：使用与模型预期使用场景一致的词汇和措辞，以促进训练字幕与期望推理提示词之间的对齐。

这些实践有助于构建高保真、任务对齐的字幕，从而提升训练效果和模型泛化能力。

### 提示词调优与评估循环

不同数据集和领域中的字幕质量差异可能非常大。不要假设一种提示词适用于所有情况；相反，应当 **根据评估结果迭代调优提示词**：

1. **从基线提示词开始** —— 以本指南中的示例为起点，并根据你的领域调整术语和结构。

2. **在小样本上生成字幕** —— 对 50–100 个有代表性的片段运行字幕生成，快速评估提示词效果。

3. **评估字幕质量** —— 根据你的质量标准，人工审查生成结果：
   - 关键场景元素（物体、主体、环境）是否被正确识别？
   - 相机视角是否分类准确？
   - 运动动态和物理交互是否被精确描述？
   - 字幕是否使用了恰当的领域术语？

4. **识别系统性错误** —— 观察字幕失败模式（例如持续遗漏某类物体、错误的相机标签、幻觉内容）。

5. **改进提示词** —— 更新 `captioning_prompt_text` 以解决已识别的问题。要尽量具体——例如，如果字幕模型经常把 dashcam 片段误分类，就在提示词中增加明确的相机角度判别标准。

6. **重新生成字幕并重新评估** —— 使用[视频重新字幕生成](#video-re-captioning)工作流，在无需重新处理视频切分的前提下迭代。重复这一过程，直到字幕达到你的质量要求。

> **关键洞察：** 最优提示词取决于你的具体数据集特征、目标使用场景和下游模型需求。通常需要经历 2–5 轮提示词迭代，才能得到可用于生产的高质量字幕。

### 高级字幕生成示例

```json
    "captioning_prompt_text": "You are a video captioning specialist tasked with creating high-quality English descriptions of traffic videos. Generate a single flowing paragraph that captures both the static scene and dynamic actions without using section headers or numbered points.\n\nBegin by analyzing the first frame to establish the scene, then smoothly transition to describing the full action sequence that follows. Your caption should be specific, precise, and focused on physical interactions.\n\nFirst Frame Analysis Requirements:\n- Camera identification:\n  * Begin caption with \"A traffic CCTV camera\" if video is from a fixed surveillance camera mounted high above ground.\n  * Begin caption with \"A stationary vehicle camera\" if video is from a non-moving camera mounted low to ground (other vehicles may be moving, but camera remains stationary).\n  * Begin caption with \"A moving vehicle camera\" if video is from a camera mounted on a moving vehicle.\n- Include other vehicles, pedestrians, cyclists, traffic lights, lane markings, traffic signs, and any other traffic related objects.\n- Include weather, time of day, road conditions, and any other environmental factors that might affect driving behavior.\n- Do NOT describe logos or watermarks visible in the video.\n- Do NOT describe vehicle number plates, street names, or building names visible in the video.\n- Do NOT describe text overlays visible in the video.\n- Do NOT describe timestamps visible in the video. Do NOT mention any dates or times in the prompt.\n- Always output in English.\n\nAction Sequence Analysis Requirements:\n- Identify and describe any of these traffic scenarios occurring in the video: accidents (collisions, near-misses), violations (wrong-way driving, jaywalking, running red lights), vehicle issues (stalled, broken down), road users (pedestrians crossing, bicyclists, motorcyclists), driving maneuvers (lane changes, U-turns, merging), traffic conditions (congestion, blocked intersions), or road hazards (debris, animals, construction). When describing detected scenarios:\n  * Focus on precise details of how the scenario unfolds, including vehicle trajectories, speeds, and physical interactions\n  * Describe the cause-and-effect relationship if visible (e.g., \"vehicle brakes suddenly due to pedestrian crossing\")\n  * Include relevant environmental factors that contribute to the scenario (wet roads, limited visibility, etc.)\n  * Use simple, direct verbs that precisely convey natural movement and physical interactions (e.g., \"accelerates,\" \"collides,\" \"swerves,\" \"rotates\" rather than \"moves\" or \"goes\")\n  * Maintain focus on physics-based descriptions rather than inferring driver intentions\n  * Ensure all descriptions flow naturally in a single cohesive paragraph without section headers",
```

> **注意**：更多字幕生成细节请参见[此文档](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/curator/REFERENCE_PIPELINES_VIDEO.md)。

<a id="video-re-captioning"></a>
### 视频重新字幕生成

在很多情况下，字幕质量需要逐步提升。此时，你可以迭代优化字幕提示词，而无需重新计算 embeddings 或 previews。下面的示例展示了一个重新字幕生成工作流：它会遍历之前切分好的视频片段，并将它们的字幕替换到一个新的输出目录中。

```json
{
    "pipeline": "split",
    "args": {
        "input_video_path": "s3://your_bucket/your_dataset/processed_data/v0/clips",
        "output_clip_path": "s3://your_bucket/your_dataset/processed_data/v1",
        "generate_embeddings": false,
        "generate_previews": false,
        "generate_captions": true,
        "splitting_algorithm": "fixed-stride",
        "fixed_stride_split_duration": 100000000,
        "captioning_algorithm": "qwen",
        "captioning_prompt_variant": "default",
        "captioning_prompt_text": "<your modified prompt here>",
        "limit": 0,
        "limit_clips": 0
   }
}
```

如果你只想重新生成字幕，而不改动已有的视频片段，可以在同一目录上重新运行 split pipeline。做法是：指定新的 `captioning_prompt_text`，同时将 `splitting_algorithm` 设为 `fixed_stride`，并给 `fixed_stride_split_duration` 赋一个很大的值。这样可以保证原始片段保持不变，因为它们通常不会超过该最大时长阈值，而新的字幕则会基于更新后的提示词重新生成。

### 分片（Sharding）

当你对切分与字幕生成结果满意后，下一步就是创建一种适合模型训练的分片数据集格式。该过程会将处理后的片段组织成 webdataset 格式。

下面是分片配置模板：

```json
{
    "pipeline": "shard",
    "args": {
        "input_clip_path": "s3://your_bucket/your_dataset/processed_data/v0",
        "output_dataset_path": "s3://your_bucket/your_dataset/webdataset/v0",
        "annotation_version": "v0",
        "verbose": true,
        "perf_profile": false
    }
}
```

#### 输出结构

分片流水线会在 `output_dataset_path` 下生成如下产物：

```
{output_dataset_path}/
├── v0/
│   ├── resolution_720/                     # all clips at 720p resolution
│       ├── aspect_ratio_16_9/              # all clips at 16:9 aspect ratio
│           ├── frames_0_255/               # all captioning windows within frames 0 to 255
│               ├── metas/                  # tar-ed .json files containing metadata for each clip
│                   ├── part_000000/
│                       ├── 000000.tar
│                       ├── 000001.tar
│               ├── t5_xxl/                 # tar-ed .pickle files for text embedding of each caption
│                   ├── part_000000/
│                       ├── 000000.tar
│                       ├── 000001.tar
│               ├── video/                  # tar-ed .mp4 files for each clip
│                   ├── part_000000/
│                       ├── 000000.tar
│                       ├── 000001.tar
│           ├── frames_256_511/             # all captioning window within frames 256 to 511
│               ├── metas/
│                   ├── part_000000/
│                       ├── 000000.tar
│               ├── t5_xxl/
│                   ├── part_000000/
│                       ├── 000000.tar
│               ├── video/
│                   ├── part_000000/
│                       ├── 000000.tar
```

<a id="running-cosmos-curator-locally"></a>
### 在本地运行 Cosmos Curator

如果你想查看一个完整的端到端示例，了解如何使用 Docker 在本地运行 Cosmos-Curator，请参考 [为 Cosmos-Predict 微调整理数据](../../recipes/data_curation/predict2_data/data_curation.md) 这一 recipe。该指南使用来自 Hugging Face 的真实数据集 [nexar_collision_prediction](https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction) 演示了完整工作流。

### SLURM 集群部署

如果要在 SLURM 集群上进行大规模处理，请参考 Cosmos-Curator 文档中的 [Launch Pipelines on SLURM](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/client/END_USER_GUIDE.md#launch-pipelines-on-slurm) 一节。该部署模式可为生产环境中的海量数据处理提供最大的计算能力。

---

## 文档信息

**发布日期：** 2025 年 10 月 9 日

### 引用

如果你使用了本内容或引用了本工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_core_curation_2025,
  title={Core Curation with Cosmos-Curator},
  author={Varghese, Jibin and Fasale, Amol and Jin, Jingyi},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/data_curation/core_curation.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用格式：**

> Jibin Varghese、Amol Fasale 与 Jingyi Jin（2025）。使用 Cosmos-Curator 进行核心数据整理。收录于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/data_curation/core_curation.html>
