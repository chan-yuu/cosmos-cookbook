# Reason 模型评估

## 标准基准测试

Cosmos Reason 模型可以使用标准化基准进行评估，以衡量其在不同场景下的推理能力。[Cosmos Reason 1 Benchmark Example](https://github.com/nvidia-cosmos/cosmos-reason1/blob/main/examples/benchmark/README.md) 提供了运行评估子集的说明，包括物理推理、空间理解和时间一致性评估。

## 在你的数据上进行自定义评估

使用你自己的视频数据（例如 robotics、egocentric）来探测任务特定的推理能力。

### 提示词模板

- “这个片段里发生了什么？”
- “请描述运动过程”
- 针对你的使用场景定制的领域特定问题

### 评估内容

- **答案正确性**：人工审核或使用 LLM-as-a-judge
- **跨时间一致性**：回答在时间维度上的连贯性
- **扎根性（Groundedness）**：是否引用了画面中真实可见的内容
- **精确性与幻觉**：这在后训练中尤为重要

## 自动指标（后训练期间）

### 指令微调（SFT）

在一个留出集上生成答案，并测量：

- **每 token loss / perplexity**：用于留出的 instruction-response 对
- **文本相似度**：与 ground-truth captions 比较的 BLEU、ROUGE、METEOR
- **嵌入相似度**：与参考答案比较的 CLIPScore、BERTScore

### 视频-字幕后训练

当使用 `<video, caption>` 对进行后训练时，请确保以下几点：

- 构建带有 ground truth 的 **MCQ/BCQ** 格式评估集。
- 跟踪模型是否随着时间推移提升了视频理解能力。
- 监控推理与理解能力的提升。

