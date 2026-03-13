# Cosmos Cookbook

[![Documentation](https://img.shields.io/badge/docs-cosmos--cookbook-blue)](https://nvidia-cosmos.github.io/cosmos-cookbook/)
[![Contributing](https://img.shields.io/badge/contributing-guide-green)](CONTRIBUTING.md)

## 中文说明

本仓库为 Cosmos Cookbook 的中文整理版本。为避免版权争议并保留原始出处，请优先参考英文原版 README：

- **Original English README**: <https://github.com/nvidia-cosmos/cosmos-cookbook/blob/main/README.md>

这是一个面向 **NVIDIA Cosmos 生态** 的综合指南。Cosmos 提供一组面向真实世界与行业场景的世界基础模型（World Foundation Models, WFMs），可用于机器人、仿真、自动驾驶系统与物理场景理解等方向。

**📚 [查看完整文档 →](https://nvidia-cosmos.github.io/cosmos-cookbook/)** —— 包含分步骤工作流、案例研究与技术配方

<https://github.com/user-attachments/assets/bb444b93-d6af-4e25-8bd0-ca5891b26276>

## 最新更新

| **日期** | **配方** | **模型** | **说明** |
|----------|----------|----------|----------|
| Mar 3 | [GR00T-Dreams: Synthetic Trajectory Generation for Robot Learning](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/end2end/gr00t-dreams/post-training.html) | Cosmos Predict 2.5, Reason 2 | 端到端的机器人合成轨迹生成流程：在 GR1 数据上后训练 Predict 2.5，生成轨迹，并使用 Cosmos Reason 2 作为视频评审器进行拒绝采样 |
| Feb 18 | [Cosmos Policy: Fine-Tuning Video Models for Visuomotor Control and Planning](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/post_training/predict2/cosmos_policy/post_training.html) | Cosmos Predict 2.5 | 配方升级至 **Cosmos Predict 2.5**：通过潜变量帧注入实现当前领先的机器人策略。结果：LIBERO 98.33%，RoboCasa **71.1%**（新 SOTA，较 Predict2 提升 4%） |
| Feb 18 | [3D AV Grounding Post-Training with Cosmos Reason 1 & 2](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/post_training/reason2/av_3d_grounding/post_training.html) | Cosmos Reason 1 & 2 | 自动驾驶场景中的 3D 车辆定位：通过 SFT（Cosmos-RL 和 Qwen-Finetune）从相机图像中检测并定位车辆 |
| Feb 4 | [Worker Safety in a Classical Warehouse](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/reason2/worker_safety/inference.html) | Cosmos Reason 2 | 在传统仓储环境中，利用上下文感知提示工程实现零样本工业安全合规与隐患检测 |
| Jan 30 | [Prompt Guide](https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/prompt_guide/reason_guide.html) | Cosmos Reason 2 | 推理提示指南 |
| Jan 29 | [Video Search and Summarization with Cosmos Reason](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/reason2/vss/inference.html) | Cosmos Reason 2 | GPU 加速的视频分析流程，支持大规模视频摘要、问答与直播告警，覆盖仓储、工厂、零售与智慧城市场景 |
| Jan 28 | [Cosmos Policy: Fine-Tuning Video Models for Visuomotor Control and Planning](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/post_training/predict2/cosmos_policy/post_training.html) | Cosmos Predict 2 | 通过潜变量帧注入实现领先的机器人策略，在 LIBERO 达到 98.5%、RoboCasa 达到 67.1%、ALOHA 达到 93.6% |
| Jan 27 | [Physical Plausibility Prediction with Cosmos Reason 2](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/post_training/reason2/physical-plausibility-check/post_training.html) | Cosmos Reason 2 | 基于 VideoPhy-2 数据集进行物理合理性预测的监督微调，提升零样本与 SFT 表现 |
| Jan 26 | [Intelligent Transportation Post-Training with Cosmos Reason 2](https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/post_training/reason2/intelligent-transportation/post_training.html) | Cosmos Reason 2 | 基于 WovenTraffic Safety 数据集后训练 Cosmos Reason 2，用于智能交通场景理解 |

## 近期活动

### NVIDIA GTC 2026

欢迎注册将于 **2026 年 3 月 16–19 日** 举办的 [NVIDIA GTC](https://www.nvidia.com/gtc/)，并将 [Cosmos 相关议程](https://www.nvidia.com/gtc/session-catalog/?sessions=S81667,CWES81669,DLIT81644,DLIT81698,S81836,S81488,S81834,DLIT81774,CWES81733,CWES81568) 添加到日历。不要错过 Jensen Huang 在 3 月 16 日（周一）上午 11:00（PT）于 SAP Center 的重磅主题演讲。

### NVIDIA Cosmos Cookoff

**[NVIDIA Cosmos Cookoff](https://luma.com/nvidia-cosmos-cookoff)** 是一个为期四周的线上 Physical AI 挑战活动，时间为 **1 月 29 日至 2 月 26 日**，面向机器人、自动驾驶与视觉 AI 开发者。

基于 NVIDIA Cosmos Reason 与 Cosmos Cookbook 配方进行构建——从第一视角机器人推理到物理合理性检查，再到交通场景感知模型，即有机会赢取 **5,000 美元**、**NVIDIA DGX Spark** 等奖项。

**[立即报名 →](https://luma.com/nvidia-cosmos-cookoff)**

赞助方：Nebius 与 Milestone。

## 前置要求

| 使用场景 | Linux (Ubuntu) | macOS | Windows |
|----------|----------------|-------|---------|
| 运行 cookbook 配方（GPU 工作流） | ✅ 支持 | ❌ | ❌ |
| 本地文档开发与贡献 | ✅ 支持 | ✅ 支持 | ⚠️ 推荐使用 WSL |

### 文档开发与贡献（全平台）

- 安装 **Git** 与 [Git LFS](#1-安装-git-lfs-必需)
- **Python** 版本 3.10+
- 用于克隆与安装依赖的网络连接

### 运行 Cookbook 配方（仅 Ubuntu）

完整的 GPU 工作流需要 Ubuntu Linux 环境与 NVIDIA GPU。

→ 完整软硬件要求请见 **[Getting Started](https://nvidia-cosmos.github.io/cosmos-cookbook/getting_started/setup.html)**。

→ 也可参考 **[Deploy on Cloud](https://nvidia-cosmos.github.io/cosmos-cookbook/getting_started/cloud_platform.html)**（Nebius、Brev 等，后续将支持更多平台）快速启动 GPU 实例。

## 快速开始

### 1. 安装 Git LFS（必需）

> ⚠️ **重要**：本仓库包含大量媒体文件（视频、图片、演示内容）。要正确克隆并使用仓库，必须安装 Git LFS。

```bash
# Ubuntu/Debian（推荐）
sudo apt update && sudo apt install git-lfs

# 全局启用 Git LFS
git lfs install
```

其他平台（macOS、Windows、Fedora）请参考官方安装指南：**[git-lfs.com](https://git-lfs.com/)**。

如果你在未启用 LFS 的情况下已经克隆仓库，可执行：

```bash
git lfs pull
```

### 2. 安装系统依赖

```bash
# 安装 uv（高性能 Python 包管理器）
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env

# 安装 just（命令运行器）
uv tool install -U rust-just
```

其他平台请参考 **[astral.sh/uv](https://astral.sh/uv/)** 的安装说明。

### 3. 克隆并初始化仓库

```bash
# 克隆仓库
git clone https://github.com/chan-yuu/cosmos-cookbook.git
cd cosmos-cookbook

# 安装依赖并初始化
just install
```

### 4. 浏览文档

```bash
# 本地启动文档
just serve-external  # 公共文档
# 或
just serve-internal  # 内部文档（如适用）
```

然后在浏览器中打开 [http://localhost:8000](http://localhost:8000)。

## 仓库结构

Cosmos Cookbook 主要由两个目录构成：

### `docs/`

包含 Markdown 文档源码：

- 技术指南与工作流
- 端到端示例与案例研究
- 分步骤配方与教程
- 入门指南

### `scripts/`

包含 cookbook 中引用的可执行脚本：

- 数据处理与整理流程
- 模型评估与质量控制脚本
- 后训练任务配置文件
- 自动化工具与实用脚本

这种结构将文档与实现分离，便于在“阅读工作流说明”和“执行对应脚本”之间快速切换。

## 媒体文件规范

贡献媒体文件时，建议优先使用 `.mp4` 而不是 `.gif`：

- **更高画质** —— MP4 支持完整色深，而 GIF 仅支持 256 色
- **更小体积** —— 现代视频编码压缩效率更高
- **支持音频** —— MP4 可在需要时包含解说音轨

请使用 **H.264** 编码，以获得最佳浏览器兼容性。

## 可用命令

```bash
# 开发
just install          # 安装依赖并初始化
just setup            # 配置 pre-commit hooks
just serve-external   # 本地启动公共文档
just serve-internal   # 本地启动内部文档

# 质量控制
just lint            # 运行 lint 与格式化
just test            # 运行全部测试与校验

# 持续集成
just ci-lint         # 运行 CI lint 检查
just ci-deploy-internal         # 部署内部文档
just ci-deploy-external         # 部署公共文档
```

## 贡献与支持

- **[贡献指南](CONTRIBUTING.md)** - 如何参与 Cosmos Cookbook
- **问题反馈**：通过 [GitHub Issues](https://github.com/chan-yuu/cosmos-cookbook/issues) 提交缺陷与功能请求
- **实践分享**：欢迎分享你使用 Cosmos 模型的创意与成果

## 许可证与联系方式

本项目会下载并安装额外的第三方开源软件。使用前请先阅读相关开源项目的许可证条款。

NVIDIA Cosmos 源代码采用 [Apache 2 License](https://www.apache.org/licenses/LICENSE-2.0) 发布。

NVIDIA Cosmos 模型采用 [NVIDIA Open Model License](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-open-model-license) 发布。如需定制许可，请联系 [cosmos-license@nvidia.com](mailto:cosmos-license@nvidia.com)。
