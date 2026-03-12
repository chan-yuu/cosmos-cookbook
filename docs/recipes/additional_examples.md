# 来自 Cosmos 模型仓库的更多示例

本页提供来自官方 Cosmos 模型仓库的推理与后训练示例链接。这些示例以更全面的模型使用与定制指南，补充了 cookbook 中的端到端教程。

## Cosmos Predict

### Cosmos Predict 2.5 *(最新)*

有关最新的 Cosmos Predict 2.5 模型文档，请访问 [Cosmos Predict 2.5 Repository](https://github.com/nvidia-cosmos/cosmos-predict2.5)。

#### 使用预训练 Cosmos Predict 2.5 模型进行推理

- **[推理指南](https://github.com/nvidia-cosmos/cosmos-predict2.5/blob/main/docs/inference.md)**：利用 Text2World、Image2World 和 Video2World 能力生成视频
- **[自动多视角推理指南](https://github.com/nvidia-cosmos/cosmos-predict2.5/blob/main/docs/inference_auto_multiview.md)**：面向自动驾驶应用的多摄像头视角生成

#### 使用 Cosmos Predict 2.5 模型进行后训练

- **[面向 DreamGen Bench 的 Video2World 后训练](https://github.com/nvidia-cosmos/cosmos-predict2.5/blob/main/docs/post-training_video2world_gr00t.md)**：使用 DreamGen benchmark 生成人形机器人轨迹

### Cosmos Predict 2

#### 使用预训练 Cosmos Predict 2 模型进行推理

- **[Text2Image 推理](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/inference_text2image.md)**：根据文本提示生成高质量图像
- **[Video2World 推理](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/inference_video2world.md)**：结合文本提示从图像/视频生成视频（单条/批量处理、多帧条件、多 GPU 推理、提示词优化、拒绝采样）
- **[Text2World 推理](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/inference_text2world.md)**：直接根据文本提示生成视频（单条/批量处理、多 GPU 推理）

#### 使用 Cosmos Predict 2 模型进行后训练

- **[Video2World 后训练指南](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/post-training_video2world.md)**：Video2World 训练系统通用指南
- **[在 Cosmos-NeMo-Assets 上进行 Video2World 后训练](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/post-training_video2world_cosmos_nemo_assets.md)**：基于 Cosmos-NeMo-Assets 数据进行后训练
- **[在鱼眼视角 AgiBotWorld-Alpha 数据集上进行 Video2World 后训练](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/post-training_video2world_agibot_fisheye.md)**：基于 AgiBotWorld-Alpha 数据集中的鱼眼视角机器人视频进行后训练
- **[在 GR00T Dreams GR1 和 DROID 数据集上进行 Video2World 后训练](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/post-training_video2world_gr00t.md)**：基于 GR00T Dreams GR1 和 DROID 数据集进行后训练
- **[在 Bridge 数据集上进行动作条件 Video2World 后训练](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/post-training_video2world_action.md)**：基于 Bridge 数据集进行动作条件后训练
- **[Text2Image 后训练指南](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/post-training_text2image.md)**：Text2Image 训练系统通用指南
- **[在 Cosmos-NeMo-Assets 上进行 Text2Image 后训练](https://github.com/nvidia-cosmos/cosmos-predict2/tree/main/documentations/post-training_text2image_cosmos_nemo_assets.md)**：基于 Cosmos-NeMo-Assets 图像数据进行后训练

## Cosmos Transfer

### Cosmos Transfer 2.5 *(最新)*

有关最新的 Cosmos Transfer 2.5 模型文档，请访问 [Cosmos Transfer 2.5 Repository](https://github.com/nvidia-cosmos/cosmos-transfer2.5)。

#### 使用预训练 Cosmos Transfer 2.5 模型进行推理

- **[推理指南](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/inference.md)**：基于深度、分割、LiDAR 和 HDMap 条件的多控制视频生成
- **[自动多视角推理指南](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/inference_auto_multiview.md)**：面向自动驾驶应用的多摄像头视角生成

#### 使用 Cosmos Transfer 2.5 模型进行后训练

- **[后训练指南](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/post-training.md)**：面向自定义控制模态与领域自适应的通用指南
- **[面向 HDMap 的自动多视角后训练](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/post-training_auto_multiview.md)**：使用 HDMap 控制的多视角自动驾驶场景

### Cosmos Transfer 1

#### 使用预训练 Cosmos Transfer 1 模型进行推理

- **[Cosmos-Transfer1-7B 推理](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b.md)**：支持多 GPU
- **[Cosmos-Transfer1-7B-Sample-AV 推理](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b_sample_av.md)**：支持多 GPU
- **[Cosmos-Transfer1-7B-4KUpscaler 推理](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b_4kupscaler.md)**：支持多 GPU 的 4K 超分辨率
- **[Cosmos-Transfer1-7B 推理（Depth）](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b_depth.md)**：基于深度的控制
- **[Cosmos-Transfer1-7B 推理（Segmentation）](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b_seg.md)**：基于分割的控制
- **[Cosmos-Transfer1-7B 推理（Edge）](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b.md#example-1-single-control-edge)**：基于边缘的控制
- **[Cosmos-Transfer1-7B 推理（Vis）](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b_vis.md)**：基于视觉的控制
- **[Cosmos-Transfer1pt1-7B 推理（Keypoint）](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1pt1_7b_keypoint.md)**：基于关键点的控制
- **[Cosmos-Transfer1-7B-Sample-AV-Multiview 推理](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/inference_cosmos_transfer1_7b_sample_av_single2multiview.md)**：多视角生成

#### 使用 Cosmos Transfer 1 模型进行后训练

- **[Cosmos-Transfer1-7B 后训练](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/training_cosmos_transfer_7b.md)**：支持多 GPU 的 Depth、Edge、Keypoint、Segmentation 和 Vis 控制
- **[Cosmos-Transfer1-7B-Sample-AV 后训练](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/training_cosmos_transfer_7B_sample_AV.md)**：支持多 GPU 的 LiDAR 和 HDMap 控制
- **[Cosmos-Transfer1-7B-Sample-AV-Multiview 后训练](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/training_cosmos_transfer_7B_sample_AV.md)**：支持多 GPU 的多视角 LiDAR 和 HDMap 控制

#### 从零开始对 Cosmos Transfer 1 模型进行后训练

- **[Cosmos-Transfer1-7B 后训练](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/training_cosmos_transfer_7b.md)**：支持多 GPU 的 Depth、Edge、Keypoint、Segmentation 和 Vis 控制
- **[Cosmos-Transfer1-7B-Sample-AV 后训练](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/training_cosmos_transfer_7B_sample_AV.md)**：支持多 GPU 的 LiDAR 和 HDMap 控制
- **[Cosmos-Transfer1-7B-Sample-AV-Multiview 后训练](https://github.com/nvidia-cosmos/cosmos-transfer1/blob/main/examples/training_cosmos_transfer_7B_sample_AV.md)**：支持多 GPU 的多视角 LiDAR 和 HDMap 控制

## Cosmos Reason 1

有关最新的 Cosmos Reason 1 模型文档，请访问 [Cosmos Reason 1 Repository](https://github.com/nvidia-cosmos/cosmos-reason1)。

### 使用 Cosmos Reason 1 模型进行后训练

- **[Cosmos Reason 1 后训练示例](https://github.com/nvidia-cosmos/cosmos-reason1/blob/main/examples/post_training/README.md)**：面向视觉语言推理任务的完整后训练指南
