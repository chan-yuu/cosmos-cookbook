# 设置与系统要求

本指南介绍如何配置环境，以运行使用 LoRA 后训练进行交通异常生成的 Cosmos Predict 2。

## 系统要求

### 最低硬件要求

- **GPU**：单节点 8 GPUs（推荐 A100、H100 或更新型号）
- **内存**：足够用于模型推理和训练的 VRAM（推荐 50GB+）
- **存储**：有足够磁盘空间存放模型权重、数据集和 checkpoints

### 软件要求

配置过程要求 Cosmos Predict 2 仓库和模型已正确安装并完成设置。

## 安装

### Cosmos Predict 2 设置

要设置 Cosmos Predict 2 仓库和模型，请按照
[Cosmos Predict 2 Setup Guide](https://github.com/nvidia-cosmos/cosmos-predict2/blob/main/documentations/setup.md)
中的安装和推理环境设置说明进行操作。

### 验证

在进入后训练流水线之前，请通过运行 Cosmos Predict 2 仓库中提供的[inference examples](https://github.com/nvidia-cosmos/cosmos-predict2?tab=readme-ov-file#user-guide)，验证模型工作正常。

## 后续步骤

完成设置和验证后，请继续阅读[后训练教程](post_training.md)，了解如何使用经过 LoRA 适配的 Cosmos Predict 2 进行交通异常生成。
