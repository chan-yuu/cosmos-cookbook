# 入门

本指南介绍为使用 Cosmos 模型而设置开发环境所需的基本工具和依赖项。这些工具为所有 Cosmos 项目的数据整理、模型后训练、评估和部署工作流奠定基础。

## 仓库设置

克隆 Cosmos Cookbook 仓库：

```shell
git clone git@github.com:nvidia-cosmos/cosmos-cookbook.git
cd cosmos-cookbook
```

### Cookbook 结构

Cosmos Cookbook 主要分为两个目录：

- **`docs/`** - 包含 Markdown 格式的源文档。其中包括构成 cookbook 内容的所有技术指南、工作流、示例和教程。

- **`scripts/`** - 包含 cookbook 中引用的所有可执行脚本。其中包括数据处理脚本、评估流水线、后训练任务的配置文件，以及各种工作流中使用的其他自动化工具。

这种结构将文档与实际实现分离，方便你在阅读工作流说明和执行相应脚本之间切换。

**注意：** 随着我们为公开发布外部仓库做准备，这些安装步骤还会继续更新。

## 前置条件

在开始之前，请确保满足以下要求。

### 硬件

运行 cookbook 配方和工作流时，至少需要 1 张 GPU 用于推理，至少需要 4 张 GPU 用于训练（推荐 8 张 GPU），并且应使用 Ampere 架构或更新架构（A100、H100）。

关于各个 Cosmos 模型（Predict1、Predict2、Transfer1 等）的具体 GPU 和内存要求，请参考 [NVIDIA Cosmos Prerequisites](https://docs.nvidia.com/cosmos/latest/prerequisites.html) 文档。

> **注意**：渲染本地文档不需要 GPU。

### 软件

- **操作系统**：Ubuntu 24.04、22.04 或 20.04
- **Python**：3.10+
- **NVIDIA Container Toolkit**：1.16.2 或更高版本
- **CUDA**：12.4 或更高版本
- **Docker Engine**
- **网络**：用于下载模型和依赖项的互联网连接

## 通用工具安装

运行 Cosmos Cookbook 需要以下系统依赖：

### pkgx

[pkgx](https://docs.pkgx.sh/) 是一个现代化的软件包管理器，可简化 CLI 工具的安装和管理。它提供隔离环境和自动依赖解析能力。

```shell
brew install pkgx || curl https://pkgx.sh | sh
```

### uv

[uv](https://docs.astral.sh/uv/) 是一个高速的 Python 包安装器和依赖解析器，可作为 pip 的即插即用替代方案。它对于管理 Cosmos 项目中的 Python 依赖至关重要。

```shell
pkgm install uv
```

#### Hugging Face CLI

[Hugging Face CLI](https://huggingface.co/docs/huggingface_hub/en/guides/cli) 是从 Hugging Face Hub 下载预训练模型 checkpoint 和数据集的重要工具。

```shell
pkgm install huggingface-cli
huggingface-cli login
```

> **注意**：你需要一个 Hugging Face 账户和访问令牌进行认证。

## 云端部署快速开始

这些云端部署指南可帮助你在无需搭建本地基础设施的情况下部署并运行 Cosmos 模型。

- **[在 Brev 上开始使用 Cosmos Reason1](brev/reason1/reason1_on_brev.md)** - 在 Brev 的云 GPU 平台上部署用于物理 AI 推理的 Cosmos Reason1。本指南涵盖资源开通、环境设置和首次推理。

- **[在 Brev 上开始使用 Transfer2.5 和 Predict2.5](brev/transfer2_5/transfer_and_predict_on_brev.md)** - 在 Brev 云基础设施上设置 Transfer2.5（视频生成）和 Predict2.5（世界预测），并提供示例工作流。
