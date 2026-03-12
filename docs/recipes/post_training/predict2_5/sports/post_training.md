# 面向体育视频生成的 LoRA 后训练

> **作者：** [Arslan Ali](https://www.linkedin.com/in/arslan-ali-ph-d-5b314239/)
> **机构：** NVIDIA

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Predict 2.5](https://github.com/nvidia-cosmos/cosmos-predict2.5) | 后训练 | 体育视频生成 |

本指南提供了使用 Cosmos Predict 2.5 模型执行 LoRA (Low-Rank Adaptation) 后训练的说明，适用于体育视频生成任务，支持 Text2World、Image2World 和 Video2World 生成模式。

## 动机

虽然基础版 Cosmos Predict 2.5 模型在通用视频生成方面表现出色，但体育内容需要对运动员动力学和比赛规则有更专业的理解。后训练可以弥补**球员运动学真实感和物理一致性**方面的关键不足，从而确保自然的人体动作和准确的球轨迹。经过适配的模型通过遵守越位线、场地边界和合法球员位置等运动特定约束，实现了更高的**规则一致性分数**。此外，后训练还显著改善了**身份一致性**，使生成序列中的球员外观、球衣号码和队伍颜色保持稳定——这对真实的体育仿真和分析应用至关重要。

## 目录

- [前置条件](#prerequisites)
- [什么是 LoRA？](#what-is-lora)
- [准备数据](#1-preparing-data)
- [LoRA 后训练](#2-lora-post-training)
  - [配置](#21-configuration)
  - [训练](#22-training)
- [使用 LoRA 后训练 checkpoint 进行推理](#3-inference-with-lora-post-trained-checkpoint)
  - [将 DCP Checkpoint 转换为 Consolidated PyTorch 格式](#31-converting-dcp-checkpoint-to-consolidated-pytorch-format)
  - [运行推理](#32-running-inference)

## 前置条件

### 1. 环境设置

请按照[设置指南](./setup.md)完成通用环境配置，包括安装依赖。

### 2. Hugging Face 配置

如果模型 checkpoints 不存在，会在后训练期间自动下载。请按如下方式配置 Hugging Face：

