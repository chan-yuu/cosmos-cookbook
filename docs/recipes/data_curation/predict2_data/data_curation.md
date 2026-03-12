# 使用 Cosmos Curator 为 Cosmos Predict 微调整理数据

> **作者：** [Hao Wang](https://www.linkedin.com/in/pkuwanghao/) • NVIDIA Cosmos Curator Team
> **组织：** NVIDIA

| **模型** | **工作负载** | **使用场景** |
|-----------|--------------|--------------|
| [Cosmos Curator](https://github.com/nvidia-cosmos/cosmos-curate) | 数据整理 | 面向 Predict 2 后训练的视频数据整理 |

## 概述

本配方演示如何使用 Cosmos Curator 为 Cosmos Predict 2 模型微调整理视频数据。你将学习如何把原始视频转换为结构化数据集，并生成语义场景切分、AI 生成字幕以及质量过滤结果——全部符合 Cosmos Predict 2 所需的数据格式。

本指南聚焦于一个可以在本地通过 Docker 运行的最小端到端工作流。我们将使用一个真实示例数据集带你逐步走完数据整理流水线。若需更高级的配置与部署选项，请参考 [Cosmos Curator Documentation](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/README.md)。

**你将学到：**

1. 使用所需模型设置 Cosmos Curator。
2. 准备源视频数据。
3. 运行整理流水线以生成可直接训练的数据集。
4. 了解高级配置选项。

## 设置 Cosmos Curator

### 克隆并安装 Cosmos Curator

这些步骤会为你提供一个 CLI，可帮助完成以下任务：

- 构建包含数据整理流水线的容器镜像。
- 从 HuggingFace 下载所需模型。
- 通过以下方式之一启动流水线：
  - 在本地使用 `docker`
  - 在 `slurm` 集群上
  - 在 `NVCF` 集群上

```bash
# Clone and install cosmos-curate
git clone --recurse-submodules https://github.com/nvidia-cosmos/cosmos-curate.git
cd cosmos-curate
uv venv --python=3.10 && source .venv/bin/activate
uv pip install poetry && poetry install --extras=local

# Verify you get the cosmos-curate CLI working
cosmos-curate --help
```

### 构建容器镜像

在构建容器镜像之前，你需要安装以下依赖：

- [Docker](https://docs.docker.com/engine/install/)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

然后即可使用以下命令构建容器镜像：

```bash
# Build the image; first build can take up to 30 minutes
cosmos-curate image build
```

### 下载模型权重

对于此配方，你需要下载以下模型：

- [TransnetV2](https://huggingface.co/Sn4kehead/TransNetV2)，用于按语义将长视频切分为短片段
- [Cosmos-Reason1-7B](https://huggingface.co/nvidia/Cosmos-Reason1-7B)，用于为片段生成字幕
- [T5-11B](https://huggingface.co/google-t5/t5-11b)，用于为字幕文本生成嵌入

```bash
cosmos-curate local launch -- pixi run python -m cosmos_curate.core.managers.model_cli download --models transnetv2,cosmos_reason1,t5_xxl
```

模型权重会下载到你的 **cosmos-curate workspace** 中，默认路径为 `${HOME}/cosmos_curate_local_workspace`。
如果你想更改 **workspace 位置**，可以定义环境变量 `COSMOS_CURATE_LOCAL_WORKSPACE_PREFIX`，将路径改为 `${COSMOS_CURATE_LOCAL_WORKSPACE_PREFIX}/cosmos_curate_local_workspace`。

## 准备源数据

下面的说明以 [nexar_collision_prediction](https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/tree/main) 数据为示例。
你需要将数据下载到 **workspace** 中。

```bash
# Go to the workspace
pushd .
cd "${COSMOS_CURATE_LOCAL_WORKSPACE_PREFIX:-$HOME}/cosmos_curate_local_workspace"

# Download a few videos into the workspace
mkdir -p nexar_collision_prediction/train/positive/
cd nexar_collision_prediction/train/positive/
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00000.mp4
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00003.mp4
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00004.mp4
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00005.mp4
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00006.mp4
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00007.mp4
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00008.mp4
wget https://huggingface.co/datasets/nexar-ai/nexar_collision_prediction/resolve/main/train/positive/00010.mp4

# let's go back
popd
```

## 运行整理流水线

```bash
cosmos-curate local launch -- pixi run python -m cosmos_curate.pipelines.video.run_pipeline split \
    --input-video-path /config/nexar_collision_prediction/train/positive/ \
    --output-clip-path /config/output-nexar/ \
    --no-generate-embeddings \
    --generate-cosmos-predict-dataset predict2 \
    --splitting-algorithm transnetv2 \
    --transnetv2-min-length-frames 120 \
    --captioning-algorithm cosmos_r1 \
    --limit 0
```

> **注意：** 如需了解所有可用流水线参数和配置选项的详细信息，请参考 [Video Pipeline Reference Documentation](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/curator/REFERENCE_PIPELINES_VIDEO.md)。

运行整理流水线时，请注意以下几点：

- `--input-video-path` 和 `--output-clip-path` 选项都要求填写 **容器内部** 的路径。
  - **workspace** 目录 `${COSMOS_CURATE_LOCAL_WORKSPACE_PREFIX:-$HOME}/cosmos_curate_local_workspace` 会挂载到容器内的 `/config`。
- 使用 `--generate-cosmos-predict-dataset predict2` 选项后，你将在指定输出路径下（即 `${COSMOS_CURATE_LOCAL_WORKSPACE_PREFIX:-$HOME}/cosmos_curate_local_workspace/output-nexar`）获得一个 `cosmos_predict2_video2world_dataset/` 目录。
  - 这正是用于[对 Cosmos Predict 2 模型进行后训练/微调](https://github.com/nvidia-cosmos/cosmos-predict2/blob/main/documentations/post-training_video2world.md#1-preparing-data)的数据集格式。
- 添加 `--transnetv2-min-length-frames 120` 选项，是为了指定片段的最小长度，因为 Cosmos Predict 2 后训练脚本要求最少帧数。
- `--limit 0` 表示不限制要处理的输入视频数量。
  - 如果系统中只有两块或更少 GPU，或者系统内存有限且输入视频过长/过多，数据整理流水线可能会因内存不足而失败并报出 OOM 错误。
  - 如果发生 OOM 错误，你可以改用较小的限制值，例如 `--limit 1`，并重复运行完全相同的命令。Cosmos Curator 能记住哪些命令已执行、哪些还未完成。

### 生成 WebDataset 格式

Cosmos Curator 还提供了另一个 `Shard-Dataset` 流水线，它会接收上述 `Split-Annotate` 流水线的输出并生成一个 webdataset。

```bash
cosmos-curate local launch -- pixi run python -m cosmos_curate.pipelines.video.run_pipeline shard \
    --input-clip-path /config/output-nexar/ \
    --output-dataset-path /config/webdataset \
    --annotation-version v0
```

同样，**workspace** 目录 `"${COSMOS_CURATE_LOCAL_WORKSPACE_PREFIX:-$HOME}/cosmos_curate_local_workspace"` 会挂载到容器中的 `/config`，
因此你可以在 `"${COSMOS_CURATE_LOCAL_WORKSPACE_PREFIX:-$HOME}/cosmos_curate_local_workspace/webdataset/v0/"` 下找到生成的 webdataset。

默认情况下，生成的 webdataset 会按分辨率、宽高比以及视频片段内的帧窗口索引进行分片。
更多细节请参见[文档](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/curator/REFERENCE_PIPELINES_VIDEO.md#shard-dataset-pipeline)。

## 高级选项

CLI 和流水线命令包含许多可配置选项。使用 `cosmos-curate --help` 或 `cosmos-curate local launch -- pixi run python -m cosmos_curate.pipelines.video.run_pipeline split --help` 可查看所有可用参数。

### 存储配置

`Cosmos Curator` 支持所有与 S3 兼容的对象存储（如 AWS、OCI、Swiftstack、GCP）。你只需正确配置 `~/.aws/credentials` 文件，并传入一个 S3 前缀。

**文档：** [Initial Setup - Storage Configuration](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/client/END_USER_GUIDE.md#initial-setup)（参见第 5 条）

### 扩展到生产平台

在只有一块 GPU 的桌面环境中本地运行，主要适合功能开发。对于生产运行，建议考虑部署到多 GPU 集群：

- [在 Slurm 上启动流水线](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/client/END_USER_GUIDE.md#launch-pipelines-on-slurm)
- [在 NVCF 上部署并启动](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/client/NVCF_GUIDE.md)
- [用于 NVCF 部署的参考 Helm Chart](https://github.com/nvidia-cosmos/cosmos-curate/tree/main/charts/cosmos-curate)
- 关于原生 K8s 部署的指南即将推出

### 流水线配置选项

#### 切分算法

- **`transnetv2`**（默认）：在场景切换处按语义切分视频。
- **`fixed-stride`**：将源视频切成固定长度的片段。

#### 字幕生成算法

- **`cosmos_r1`**（默认）：使用 Cosmos-Reason1-7B 模型。
- **`qwen`**：使用 Qwen 2.5 VL 模型（需要在模型下载列表中加入 `qwen2.5_vl`）。
- **`phi4`**：使用 Phi-4 模型（需要在模型下载列表中加入 `phi_4`）。

#### 自定义字幕提示词

除了默认提示词外，你也可以传入自定义提示词：

```bash
--captioning-prompt-text 'Describe the driving scene in detail, focusing on road conditions and vehicle behavior.'
```

#### 质量过滤器

通过过滤低质量片段来提升数据集质量：

- **运动过滤器**：`--motion-filter enable`（移除静止/低运动片段）
- **美学过滤器**：`--aesthetic-threshold 3.5`（过滤低质量或模糊帧）
- **基于 VLM 的过滤器**：`--qwen-filter enable`（语义过滤；需要在模型下载列表中加入 `qwen2.5_vl`）

**文档：** [Split-Annotate Pipeline Reference](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/curator/REFERENCE_PIPELINES_VIDEO.md#split-annotate-pipeline-stages)

### 扩展流水线

如果你需要添加自定义处理阶段（例如新的过滤逻辑、自定义标注），请参考 [Pipeline Design Guide](https://github.com/nvidia-cosmos/cosmos-curate/blob/main/docs/curator/PIPELINE_DESIGN_GUIDE.md)。

---

## 文档信息

**发布日期：** 2025 年 10 月 29 日

### 引用

如果你使用了此配方或参考了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_curate_data_for_2025,
  title={Curate data for Cosmos Predict Fine-Tuning using Cosmos Curator},
  author={Wang, Hao},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/data_curation/predict2_data/data_curation.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Hao Wang（2025）。使用 Cosmos Curator 为 Cosmos Predict 微调整理数据。载于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/data_curation/predict2_data/data_curation.html>
