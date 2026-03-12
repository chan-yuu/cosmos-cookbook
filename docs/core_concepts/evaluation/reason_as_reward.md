# Cosmos Reason 作为奖励模型

## 概览

NVIDIA Cosmos Reason 是一个开放、可定制、参数规模为 7B 的推理型 vision language model（VLM），面向 Physical AI 和机器人领域。本文档介绍如何将 Cosmos Reason 模型用于视频评估，主要包含两种模式：

1. **奖励模型（Reward Model）**：为 RL 训练和模型选择对视频打分。
2. **视频评论器（Video Critic）**：提供详细分析和结构化反馈。

## 模型能力

- **物理理解**：重力、碰撞、流体动力学、物体恒存性
- **时空推理**：三维关系与运动一致性
- **具身 AI 评估**：智能体行为与环境交互
- **思维链分析**：无需人工标注即可进行逐步推理
- **零样本评估**：适用于多种领域与场景

## 安装与设置

### 第 1 步：安装依赖

```bash
# Install core dependencies
pip install torch torchvision transformers
pip install mediapy numpy pillow qwen-vl-utils huggingface-hub

# Verify installation
python -c "import torch; print(f'PyTorch: {torch.__version__}'); print(f'CUDA available: {torch.cuda.is_available()}')"
```

### 第 2 步：下载模型

```bash
# Download Cosmos Reason 1-7B-Reward model
huggingface-cli download nvidia/Cosmos-Reason1-7B-Reward --local-dir ./checkpoints --token <YOUR_HF_TOKEN>
```

> **注意**：需要 HuggingFace 账号，以及具有 `nvidia/Cosmos-Reason1-7B-Reward` 模型访问权限的 token。

## 奖励模型用法

### 单视频评估

```bash
python inference.py --video path/to/video.mp4 --checkpoint ./checkpoints
```

**输出：**

```
Video: sample_video.mp4
Physical accuracy: No
Score (high is good): 0.2341
```

### 批处理

```bash
# Process directory of videos
python batch_inference.py --checkpoint ./checkpoints --video-dir ./test_videos --output-dir ./results
```

### 评分体系

- **分数范围**：0.0 到 1.0（越高表示物理准确性越好）
- **二元分类**：`Yes` = 检测到异常，`No` = 未检测到异常
- **阈值**：高（0.7-1.0）、中（0.3-0.7）、低（0.0-0.3）

### 物理评估框架

#### 模型评估什么

- **重力**：是否符合真实的重力行为
- **碰撞**：物体交互的物理过程是否合理
- **物体交互**：是否存在合理的因果关系
- **流体动力学**：液体和气体行为是否真实
- **物体恒存性**：物体存在是否前后一致
- **人体运动**：身体动作与关节约束是否自然

#### 模型忽略什么

- 动画风格（卡通风格不会自动被判定为异常）
- 音频内容
- 光照、阴影、相机效果
- 艺术风格和背景元素

## 视频评论器用法

### 详细分析模式

[Cosmos Reason 1 Video Critic Example](https://github.com/nvidia-cosmos/cosmos-reason1/blob/main/examples/benchmark/README.md) 提供了结构化视频分析示例：

```bash
# Run video critic evaluation
python video_critic.py \
    --video path/to/video.mp4 \
    --checkpoint ./checkpoints \
    --critique_mode detailed \
    --output_format structured
```

### 评论器输出格式

```json
{
  "video_analysis": {
    "physical_accuracy": {
      "score": 0.85,
      "violations": ["gravity_inconsistency_at_12s"],
      "explanation": "Object falls upward at timestamp 12 seconds"
    },
    "reasoning_chain": {
      "logical_consistency": 0.92,
      "causal_relationships": "strong",
      "temporal_coherence": "maintained"
    },
    "content_quality": {
      "visual_coherence": 0.88,
      "object_permanence": 0.95,
      "scene_understanding": "excellent"
    }
  }
}
```

### 视频评论器能力

- **物理合理性评估**：详细分析物理规律违规情况
- **推理链分析**：逐步拆解逻辑一致性
- **内容质量点评**：评估视觉一致性与时间一致性
- **上下文理解**：评估场景上下文与物体关系理解

## 高级配置

### 自定义提示词工程

通过修改 `inference.py` 中的提示词，自定义评估重点：

```python
# Focus on specific physics
USER_PROMPT = "Focus on fluid dynamics and ignore human motion artifacts"

# Domain-specific evaluation
SYSTEM_PROMPT = "Evaluate this medical procedure video for physical plausibility"

# Multi-step reasoning
USER_PROMPT = "Provide step-by-step analysis of physical anomalies with explanations"
```

### 领域适配

**医疗视频**：解剖准确性与医疗操作真实感  
**机器人**：机械约束与机器人行为  
**合成数据**：仿真物理与渲染准确性  
**游戏**：游戏物理与角色运动真实感

## 与其他指标结合使用

### 综合评估流水线

```bash
# Combine multiple evaluation approaches
python comprehensive_evaluation.py \
    --videos ./test_videos/*.mp4 \
    --metrics fid,fvd,sampson \
    --cosmos_reward ./checkpoints \
    --cosmos_critic ./checkpoints \
    --output_report ./evaluation_report.json
```

### 使用场景

#### 奖励模型应用

- 强化学习训练信号
- 模型选择与 checkpoint 排序
- 生成内容的质量过滤
- 大规模自动化评估

#### 视频评论器应用

- 生成视频质量控制
- 训练数据整理与过滤
- 基准评估与模型对比
- 研究分析与消融实验

## 可视化与分析

### Streamlit 界面

```bash
cd experimental/afasale/visualization

# Launch interactive results browser
./run_streamlit.sh ./results/video ./results/text
```

**功能：**

- 交互式视频浏览与分数查看
- 评估结果的统计分析
- 异常检测与问题视频识别
- 不同模型之间的对比分析

## 性能优化

### 系统要求

- **GPU**：建议使用支持 CUDA 的 GPU
- **内存**：批处理大约需要 15GB VRAM
- **处理时间**：每个视频约 10-30 秒
- **存储**：需要足够空间存放输出文件与日志

### 优化建议

```bash
# Large datasets
python batch_inference.py --checkpoint ./checkpoints --video-dir ./dataset --batch-size 4

# Memory-constrained environments
python inference.py --video input.mp4 --checkpoint ./checkpoints --low-memory-mode

# Multi-GPU processing
python distributed_inference.py --checkpoint ./checkpoints --video-dir ./dataset --num-gpus 4
```

## 最佳实践

### 评估工作流

1. **预处理**：确保视频采用受支持的格式（MP4、AVI、MOV）。
2. **验证**：先在已知的好/坏样本上测试。
3. **批处理**：大规模数据集请使用 batch inference。
4. **自定义提示词**：针对特定领域调整评估标准。
5. **结果分析**：同时查看分数与详细评论结果。
6. **组合使用**：与其他指标配合，获得更全面的评估。

### 质量保证

- **提示词工程**：记录并进行版本管理自定义提示词。
- **结果验证**：人工核验一部分结果用于校准。
- **可复现性**：使用一致的 checkpoint 与提示词。
- **文档记录**：跟踪评估配置和结果。

## 资源

### 官方文档

- [Cosmos Reason 1 GitHub Repository](https://github.com/nvidia-cosmos/cosmos-reason1)
- [Cosmos Reason 1-7B-Reward Model](https://huggingface.co/nvidia/Cosmos-Reason1-7B-Reward)

### 示例与教程

- [Benchmark Example](https://github.com/nvidia-cosmos/cosmos-reason1/blob/main/examples/benchmark/README.md)
- [Video Critic Example](https://github.com/nvidia-cosmos/cosmos-reason1/blob/main/examples/video_critic/README.md)

### 其他资源

- [Physical AI and Robotics Documentation](https://research.nvidia.com/labs/dir/)
- [VLM Evaluation Best Practices](https://research.nvidia.com/vlm-evaluation)

## 引用

在研究中使用 Cosmos Reason 模型时，请引用相应论文，并注明 NVIDIA Cosmos 项目。

