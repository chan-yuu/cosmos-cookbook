# 设置与系统要求

本指南介绍如何设置环境，以运行 **Cosmos Transfer 2.5 + FiftyOne** 工作流，对 **BioTrove moth dataset** 进行增强，并在 FiftyOne App 中探索结果。

设置主要分为三部分：

1. 系统与软件要求
2. 安装 Cosmos Transfer 2.5 及其依赖项
3. 安装并配置 FiftyOne 和数据集

---

## 系统要求

### 最低硬件要求

- **GPU**:
  - 1 张或多张 NVIDIA GPU
  - **至少 80 GB GPU memory (VRAM)**（例如 A100 80GB、H100 80GB）
  - Ampere 架构或更新版本（推荐 RTX 30 Series、A100、H100 或更高版本）
- **存储**:
  - **至少 100 GB 可用磁盘空间**，用于：
    - Cosmos Transfer 2.5 仓库和模型权重
    - FiftyOne 数据集及派生视频/图像（边缘图、输出、最后一帧）

### 支持的平台

- **操作系统**: Linux x86-64
  - 推荐：**Ubuntu ≥ 22.04**（glibc ≥ 2.35）
- **NVIDIA Driver**:
  - **≥ 570.124.06**，兼容 CUDA **12.8.1**（或 CUDA 12+）
- **Python**:
  - **Python 3.10**（与 Cosmos Transfer 2.5 要求一致）

---

## 软件要求

你需要以下软件组件：

- **CUDA Toolkit**，需与你的驱动兼容（CUDA 12+）
- **PyTorch ≥ 2.5**，且带有 CUDA 支持
- **TorchVision**
- **Git**
- **FFmpeg**（CLI）— 用于将图像转换为视频以及处理视频 I/O
- **Python 包**:
  - `fiftyone`
  - `opencv-python`（用于 Canny 边缘和视频处理）
  - Cosmos Transfer 2.5 Python 包（从仓库安装）
  - 其他 Cosmos 依赖项（通常通过其设置指南安装），例如：
    - `json5`
    - `gradio`（可选 UI）
    - `easyio`（用于多存储后端）

> 如需获取最准确的 Cosmos Transfer 2.5 依赖项列表，请始终参考官方 [**Cosmos Transfer 2.5 设置指南**](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/setup.md)。

---

## 安装

### 1. 创建并激活 Python 环境

你可以使用 `conda` 或 `venv`。以下是使用 `conda` 的示例：

```bash
conda create -n cosmos-transfer2_5-biotrove python=3.10 -y
conda activate cosmos-transfer2_5-biotrove
```

或者使用 `venv`：

```bash
python3.10 -m venv cosmos-transfer2_5-biotrove
source cosmos-transfer2_5-biotrove/bin/activate
```

---

### 2. 安装 FFmpeg

在 Ubuntu 上：

```bash
sudo apt-get update
sudo apt-get install -y ffmpeg
```

你应该能够运行：

```bash
ffmpeg -version
```

且不会报错。

---

### 3. 安装 FiftyOne 和配套 Python 包

安装 FiftyOne 和核心 Python 依赖项：

```bash
pip install fiftyone opencv-python
```

可选但推荐，用于配合 notebook 和可视化：

```bash
pip install jupyterlab umap-learn
```

---

### 4. 克隆并安装 Cosmos Transfer 2.5

克隆 Cosmos Transfer 2.5 仓库，并以可编辑模式安装；同时按照 [**Cosmos Transfer 2.5 设置指南**](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/setup.md) 完成环境配置和模型权重下载。

---

### 5. 配置环境变量与路径

设置 `COSMOS_DIR`：

```bash
export COSMOS_DIR=/path/to/cosmos-transfer2.5
```

可选：

```bash
export LIST_FILE=/path/to/video_list.txt
export MAX_VIDS=100
```

---

## 数据集设置（BioTrove Moth Dataset）

此配方使用来自 Hugging Face Hub 的 BioTrove 数据集子集：

```python
dataset_src = fouh.load_from_hub(
    "pjramg/moth_biotrove",
    persistent=True,
    overwrite=True,
    max_samples=2,
)
```

---

## 验证

### 验证 Cosmos Transfer 2.5

```bash
cd $COSMOS_DIR
python examples/inference.py --help
```

### 验证 FiftyOne + FFmpeg

```python
import fiftyone as fo
print("FiftyOne version:", fo.__version__)

dataset_src = fouh.load_from_hub(
    "pjramg/moth_biotrove",
    persistent=True,
    overwrite=False,
    max_samples=2,
)
```

最小 FFmpeg 测试：

```bash
ffmpeg -loop 1 -i some_image.jpg -t 1 -c:v libx264 -pix_fmt yuv420p test.mp4
```

---

## 后续步骤

你可以前往[推理教程](inference.md)完成 Cosmos Transfer 2.5 + FiftyOne 工作流。也可以访问这个[教程](https://docs.voxel51.com/tutorials/cosmos-transfer-integration.html)，直接在你的环境中运行它。
