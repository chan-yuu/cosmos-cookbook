# 环境设置指南

本指南介绍 [GR00T-Dreams 后训练配方](post-training.md) 的环境准备要求。请按照各自仓库的官方安装说明安装并配置 **Cosmos Predict 2.5** 和 **Cosmos Reason 2**；本页仅总结所需内容，并补充与本配方相关的额外步骤。

## 系统要求

有关详细的硬件和软件要求，请参阅下方链接的官方指南。简要来说：

* NVIDIA GPU，需为 Ampere 架构（RTX Pro 6000、A100、H100）或更新型号
* 与 CUDA 12.8+ 兼容的 NVIDIA 驱动（精确版本请参见 [Predict 2.5](https://github.com/nvidia-cosmos/cosmos-predict2.5) / [Reason 2](https://github.com/nvidia-cosmos/cosmos-reason2)）
* Linux x86-64
* glibc>=2.35（例如 Ubuntu >=22.04）
* Python 3.10

## 安装

### 1. 克隆仓库

```shell
git clone https://github.com/nvidia-cosmos/cosmos-predict2.5.git
git clone https://github.com/nvidia-cosmos/cosmos-reason2.git
```

### 2. 安装 Cosmos Predict 2.5

请遵循 Cosmos Predict 2.5 的**官方安装说明**（环境、依赖、CUDA 变体）。不要依赖本页获取精确命令。

```shell
cd cosmos-predict2.5
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
uv sync --extra=cu128
source .venv/bin/activate
```

* **[Cosmos Predict 2.5 — Installation](https://docs.nvidia.com/cosmos/latest/predict2.5/installation.html)**（NVIDIA Docs）
* 或者参阅 [Cosmos Predict 2.5 repository README](https://github.com/nvidia-cosmos/cosmos-predict2.5)，了解 clone、`uv` 和 `uv sync` 步骤。

### 3. 安装 Cosmos Reason 2

请遵循 Cosmos Reason 2 的**官方安装说明**（环境、依赖、CUDA 变体）。

```shell
cd ../cosmos-reason2
uv sync --extra=cu128 --active --inexact
source .venv/bin/activate
```

* **[Cosmos Reason 2 — Repository README](https://github.com/nvidia-cosmos/cosmos-reason2)**（包含 clone、`uv`、`uv sync` 和推理设置）
* 若需针对后训练的专用设置（本配方可选），请参阅 [Cosmos Reason 2 Post-Training Installation](https://github.com/nvidia-cosmos/cosmos-reason2/blob/main/examples/cosmos_rl/README.md)。

完成两个仓库的安装后，你应能按照各自文档中的说明，在对应仓库根目录运行推理。

---

* `--extra=cu128`：CUDA 12.8
* `--extra=cu129`：CUDA 12.9

## 配方专用配置

以下配置专用于本 GR00T-Dreams 工作流，并不能替代官方安装步骤。

### 下载 checkpoints（Cosmos Predict 2.5）

1. 获取一个具有 `Read` 权限的 [Hugging Face Access Token](https://huggingface.co/settings/tokens)
2. 安装 [Hugging Face CLI](https://huggingface.co/docs/huggingface_hub/en/guides/cli)：`uv tool install -U "huggingface_hub[cli]"`
3. 登录：运行 `hf auth login`，并输入第 1 步创建的 token。
4. 接受 [NVIDIA Open Model License Agreement](https://huggingface.co/nvidia/Cosmos-Predict2.5-2B)。

checkpoints 会在推理和后训练期间自动下载。若要更改缓存位置，请设置 [HF_HOME](https://huggingface.co/docs/huggingface_hub/en/package_reference/environment_variables#hfhome)。

> **💡 提示：** 请确保 `HF_HOME` 有足够的磁盘空间。

### 训练输出目录

设置 Cosmos Predict 2.5 保存 checkpoints 和产物的位置（默认：`/tmp/imaginaire4-output`）：

```bash
export IMAGINAIRE_OUTPUT_ROOT=/path/to/your/output/directory
```

> **💡 提示：** 请确保 `IMAGINAIRE_OUTPUT_ROOT` 有足够的磁盘空间。

### Weights & Biases (W&B) 日志

默认情况下，训练会尝试将指标记录到 Weights & Biases。你有以下几种选择：

#### 选项 1：启用 W&B

如果要启用完整的 W&B 实验追踪：

1. 在 [wandb.ai](https://wandb.ai) 创建一个免费账户
2. 从 [https://wandb.ai/authorize](https://wandb.ai/authorize) 获取你的 API key
3. 设置环境变量：

    ```bash
    export WANDB_API_KEY=your_api_key_here
    ```

> ⚠️ **安全警告：** 请将 API key 存储在环境变量或安全密钥库中。切勿将 API key 提交到源代码管理系统。

#### 选项 2：禁用 W&B

在训练命令中添加 `job.wandb_mode=disabled`，即可禁用 wandb 日志记录。

## 下一步

完成设置后，请继续阅读[后训练教程](post-training.md)，了解如何在 Gr00t Trajectories 上训练 Cosmos Predict 2.5。
