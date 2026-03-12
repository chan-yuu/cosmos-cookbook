# 安装与系统要求

本指南介绍了运行 Cosmos Transfer 1 以对 ITS 图像进行天气增强所需的设置要求。

## 系统要求

### 最低硬件要求

- **GPU**: 1 张或更多 GPU（推荐 A100、H100 或更新型号）
- **Memory**: 用于模型推理的充足显存
- **Storage**: 用于模型权重的足够磁盘空间

### 软件要求

该设置要求已正确安装并配置 Cosmos Transfer 1 仓库和模型。

## 安装

### Cosmos Transfer 1 设置

要设置 Cosmos Transfer 1 仓库和模型，请按照以下详细安装与推理设置说明操作：

**[Cosmos Transfer 1 安装指南](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/INSTALL.md#inference)**

该安装指南提供了以下内容的完整步骤：

- 仓库克隆与设置
- 环境配置
- 模型权重下载
- 依赖安装
- 推理配置

### 验证

完成安装后，请运行 Cosmos Transfer 1 仓库中提供的推理示例来验证设置，确保模型工作正常，然后再继续进行天气增强流程。

## 后续步骤

设置完成后，请继续阅读[推理教程](inference.md)，了解如何使用 Cosmos Transfer 1 对 ITS 图像进行天气增强。
