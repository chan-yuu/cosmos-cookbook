# 设置与系统要求

本指南介绍运行 Cosmos Predict2 生成合成 ITS 图像所需的设置要求。

## 系统要求

### 最低硬件要求

- **GPU**：采用 Ampere 架构（RTX 30 系列、A100）或更新架构的 NVIDIA GPU
- **内存**：满足模型推理所需的显存容量
- **存储**：具备足够的模型权重存储空间

### 软件要求

本设置要求已正确安装并配置 Cosmos Predict2 仓库和模型。

## 安装

### Cosmos Predict2 设置

如需设置 Cosmos Predict2 仓库和环境，请按照官方设置说明进行操作：

**[Cosmos Predict2 设置指南](https://github.com/nvidia-cosmos/cosmos-predict2/blob/main/documentations/setup.md)**

该设置指南包含以下步骤：

- 克隆并设置仓库
- 配置环境
- 下载模型权重
- 安装依赖项
- 通过 Docker 容器安装
- 配置推理

### 验证

完成安装后，请运行 Cosmos Predict2 文档中的 text-to-image 推理演练来验证设置，确保在继续 ITS 工作流之前，端到端推理可以正常运行：

参见：[Cosmos Predict2 Text-to-Image 推理](https://github.com/nvidia-cosmos/cosmos-predict2/blob/main/documentations/inference_text2image.md)

## 后续步骤

设置完成后，请继续阅读[推理教程](inference.md)，了解如何使用 Cosmos Predict2 进行 ITS 图像合成/推理。
