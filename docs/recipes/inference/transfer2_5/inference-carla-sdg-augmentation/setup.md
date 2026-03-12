# 设置与系统要求

本指南介绍运行 Cosmos Transfer 2.5 对来自模拟器的合成数据进行视频增强所需的设置要求。

## 系统要求

### 最低硬件要求

- **GPU**：1 张或多张 GPU（推荐 A100、H100 或更高版本）
- **内存**：具备足够的 VRAM 以进行模型推理
- **存储**：具备足够的磁盘空间以存放模型权重

### 软件要求

该设置要求正确安装并配置 Cosmos Transfer 2.5 仓库和模型。

## 安装

### Cosmos Transfer 2.5 设置

要设置 Cosmos Transfer 2.5 仓库和模型，请遵循 [Cosmos Transfer 2.5 设置指南](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/setup.md)。

安装指南提供了以下完整步骤：

- 克隆并设置仓库
- 配置环境
- 下载模型权重
- 安装依赖项
- 配置推理

### 验证

完成安装后，请运行 Cosmos Transfer 2.5 仓库中提供的[推理示例](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/inference.md)来验证设置，确保模型在继续进行模拟器增强流水线之前能够正常工作。

## 后续步骤

设置完成后，请继续阅读[推理教程](inference.md)，了解如何使用 Cosmos Transfer 2.5 进行 Sim2Real 数据增强。
