# 安装与系统要求

本指南介绍了运行 Cosmos Transfer 1 进行仓库多视角推理以及面向机器人导航任务推理所需的设置要求。

## 系统要求

### 最低硬件要求

- **GPU**: 1 张或更多 GPU（推荐 A100、H100 或更新型号）
- **Memory**: 用于模型推理的充足显存
- **Storage**: 用于模型权重的足够磁盘空间

### 软件要求

该设置要求已正确安装并配置 Cosmos Transfer 1 仓库以及 Cosmos Cookbook。

## 安装

### Cosmos Cookbook 设置

按照以下完整设置教程来配置 Cosmos Cookbook 仓库及其依赖：

**[入门指南](../../../../getting_started/setup.md)**

入门指南详细说明了以下内容：

- 仓库克隆与设置
- 环境配置
- 必要工具安装（uv、Hugging Face CLI、AWS CLI 等）
- 开发依赖

### Cosmos Transfer 1 设置

要设置 Cosmos Transfer 1 仓库和模型，请参阅 [Cosmos Transfer 1 Installation Guide](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/INSTALL.md#inference) 获取详细的安装与推理设置说明。

该安装指南提供了以下内容的完整步骤：

- 仓库克隆与设置
- 环境配置
- 模型权重下载
- 依赖安装
- 推理配置

完成安装后，请将 cookbook 示例资源复制到你的 Cosmos Transfer 1 仓库中。假设你的 Cosmos Transfer 1 仓库根目录通过 `$COSMOS_TRANSFER_ROOT` 可用，请运行以下命令：

```bash
cp -r scripts/examples/transfer1/* "$COSMOS_TRANSFER_ROOT/examples/cookbook/"
```

将 `$COSMOS_TRANSFER_ROOT` 替换为 Cosmos Transfer 1 目录的实际路径，或者将该路径导出为环境变量。

#### 安装 ffmpeg（视频预处理/后处理所必需）

将 ffmpeg 安装到当前激活的 Conda 环境中：

```bash
conda install -c conda-forge ffmpeg
```

## 后续步骤

设置完成后，请继续阅读[推理教程](inference.md)，了解如何使用 Cosmos Transfer 1 进行仓库多视角推理。
