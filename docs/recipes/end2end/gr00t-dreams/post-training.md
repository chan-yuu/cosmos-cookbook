# 利用世界基础模型为机器人学习生成合成轨迹

> **作者：** [Rucha Apte](https://www.linkedin.com/in/ruchaa-apte/), [Jingyi Jin](https://www.linkedin.com/in/jingyi-jin/), [Saurav Nanda](https://www.linkedin.com/in/sauravnanda/)
> **组织：** NVIDIA

| **模型**                                                                | **工作负载**   | **使用场景** |
| ------------------------------------------------------------------------ | -------------- | ---------------------------------------------- |
| [Cosmos Predict 2.5](https://github.com/nvidia-cosmos/cosmos-predict2.5) | 后训练、推理 | 合成轨迹生成 |
| [Cosmos Reason 2](https://github.com/nvidia-cosmos/cosmos-reason2)       | 推理 | 对合成轨迹进行推理与筛选 |

本指南将带你在 [PhysicalAI-Robotics-GR00T-GR1](https://huggingface.co/datasets/nvidia/PhysicalAI-Robotics-GR00T-GR1) 开放数据集上对 Cosmos Predict 2.5 模型进行后训练，以便为机器人学习应用生成合成机器人轨迹。完成后训练后，我们将使用微调后的模型在 [PhysicalAI-Robotics-GR00T-Eval](https://huggingface.co/datasets/nvidia/PhysicalAI-Robotics-GR00T-Eval) 数据集上生成轨迹预测。最后，我们会利用 Cosmos Reason 2 通过评估这些生成轨迹的物理合理性来对其进行打分，从而量化并筛选出有效、真实且成功的机器人动作。
整个流程包含下列步骤：

<p align="center">
  <img src="assets/Gr00t-Dreams.png" alt="GR00T Dreams" width="900">
</p>

## 动机

通用机器人技术正在兴起，驱动力来自机电系统与机器人基础模型的进步，但技能学习的规模化仍然受限于海量训练数据的需求。建立在 [NVIDIA Cosmos](https://www.nvidia.com/en-us/ai/cosmos/?sortBy=developer_learning_library%2Fsort%2Ffeatured_in.cosmos%3Adesc%2Ctitle%3Aasc&hitsPerPage=6) 之上的 [NVIDIA Isaac GR00T-Dreams](https://github.com/nvidia/gr00t-dreams) 能够根据单张图像和语言提示生成大规模合成轨迹数据，以此缓解这一瓶颈。这使得像 [NVIDIA Isaac GR00T N1.5](https://developer.nvidia.com/isaac/gr00t) 这样的模型能够更高效地用于推理和技能学习。
*世界基础模型* 可以根据当前观测和语言，对未来世界状态（如视频或轨迹）进行预测或推理。在本配方中，Cosmos Predict 2.5 充当生成合成轨迹的世界模型，而 Cosmos Reason 2 则负责评估这些轨迹是否遵循物理规律。

## 目录

- [先决条件](#prerequisites)
- [准备数据](#1-preparing-data)
- [后训练 Cosmos Predict](#2-post-training-cosmos-predict)
- [使用后训练后的 Cosmos Predict 进行推理](#3-inference-with-post-trained-cosmos-predict)
  - [将 DCP Checkpoint 转换为 Consolidated PyTorch Format](#31-converting-dcp-checkpoint-to-consolidated-pytorch-format)
  - [运行推理](#32-running-inference)
- [使用 Cosmos Predict 进行策略评估](#4-policy-evaluation-using-cosmos-predict)
- [将 Cosmos-Reason 作为视频评论器进行拒绝采样](#5-using-cosmos-reason-as-video-critic-for-rejection-sampling)
- [计算指标](#6-computing-metrics)
- [局限性与注意事项](#limitations-and-considerations)
- [结论](#7-conclusion)

<a id="prerequisites"></a>
## 先决条件

请先阅读[环境设置指南](./setup.md)，完成通用环境准备，包括安装 Cosmos Predict 2.5 和 Cosmos Reason 2 所需的依赖。

<a id="1-preparing-data"></a>
## 1. 准备数据

本节以及直到 **多视频 rollout 生成**（第 4 节）为止的所有步骤，都默认你位于 **cosmos-predict2.5** 仓库根目录，除非某一步另有说明（例如 Cosmos Reason 2 的步骤需要在 cosmos-reason2 根目录下执行）。
首先，我们将下载 [GR1 训练数据集](https://huggingface.co/datasets/nvidia/PhysicalAI-Robotics-GR00T-GR1)，然后对其进行预处理，为每个视频创建文本提示 `txt` 文件。

### 下载 DreamGen Bench 训练数据集

```bash
cd cosmos-predict2.5
```

```bash
hf download nvidia/GR1-100 --repo-type dataset --local-dir datasets/benchmark_train/hf_gr1/ && \
mkdir -p datasets/benchmark_train/gr1/videos && \
mv datasets/benchmark_train/hf_gr1/gr1/*mp4 datasets/benchmark_train/gr1/videos && \
mv datasets/benchmark_train/hf_gr1/metadata.csv datasets/benchmark_train/gr1/
```

### 预处理 DreamGen Bench 训练数据集

```bash
python -m scripts.create_prompts_for_gr1_dataset --dataset_path datasets/benchmark_train/gr1
```

运行上述预处理后，数据集目录结构应如下所示：

```bash
datasets/benchmark_train/gr1/
├── metas/
│   ├── *.txt
├── videos/
│   ├── *.mp4
├── metadata.csv
```

### 训练数据集预览

| 输入提示 | 视频文件 |
| ----------------- | ---------- |
| 机器人手臂正在执行任务。请用右手将绿色小白菜从浅棕色桌子右侧拿起，放到金属篮底层。 | <video width="320" controls autoplay loop muted><source src="assets/1.mp4" type="video/mp4"></video> |
| 机器人手臂正在执行任务。请用右手将魔方从架子顶层拿起，放到架子底层。 | <video width="320" controls autoplay loop muted><source src="assets/2.mp4" type="video/mp4"></video> |
| 机器人手臂正在执行任务。请用右手将香蕉从青绿色盘子拿起，放到木桌上。 |  <video width="320" controls autoplay loop muted><source src="assets/3.mp4" type="video/mp4"></video> |

<a id="2-post-training-cosmos-predict"></a>
## 2. 后训练 Cosmos Predict

在本教程中，我们将对 Cosmos-Predict2.5 2B 模型进行后训练。14B 的后训练过程与下面的 2B 示例非常相似。
运行以下命令，使用 GR1 数据执行一个示例后训练任务。

```bash
torchrun --nproc_per_node=1 --master_port=12341 -m scripts.train --config=cosmos_predict2/_src/predict2/configs/video2world/config.py -- experiment=predict2_video2world_training_2b_groot_gr1_480
```

> **注意**：如果要禁用 W&B 日志，请在上述命令中添加 `job.wandb_mode=disabled`

该脚本使用 `predict2_video2world_training_2b_groot_gr1_480` 配置。请查看下面的 job config 以了解其定义方式。

```bash
predict2_video2world_training_2b_groot_gr1_480 = dict(
    ...,
    dataloader_train=dataloader_train_gr1,
    ...,
    job=dict(
        project="cosmos_predict_v2p5",
        group="video2world",
        name="2b_groot_gr1_480",
    ),
    ...,
)
```

> checkpoints 会保存到 ${IMAGINAIRE_OUTPUT_ROOT}/PROJECT/GROUP/NAME/checkpoints。默认情况下，IMAGINAIRE_OUTPUT_ROOT 为 /tmp/imaginaire4-output。在上述示例中，PROJECT 为 `cosmos_predict_v2p5`，GROUP 为 `video2world`，NAME 为 `2b_groot_gr1_480`。checkpoints 将以 Distributed Checkpoint (DCP) 格式保存。

**目录结构示例：**

```
checkpoints/
├── iter_{NUMBER}/
│   ├── model/
│   │   ├── .metadata
│   │   └── __0_0.distcp
│   ├── optim/
│   ├── scheduler/
│   └── trainer/
└── latest_checkpoint.txt
```

<a id="3-inference-with-post-trained-cosmos-predict"></a>
## 3. 使用后训练后的 Cosmos Predict 进行推理

<a id="31-converting-dcp-checkpoint-to-consolidated-pytorch-format"></a>
### 3.1 将 DCP Checkpoint 转换为 Consolidated PyTorch Format

由于训练期间 checkpoints 以 DCP 格式保存，因此在推理前需要先将其转换为 Consolidated PyTorch 格式（`.pt`）。请使用 `convert_distcp_to_pt.py` 脚本：

```bash
# Get path to the latest checkpoint
CHECKPOINTS_DIR=${IMAGINAIRE_OUTPUT_ROOT:-/tmp/imaginaire4-output}/cosmos_predict_v2p5/video2world/2b_groot_gr1_480/checkpoints
CHECKPOINT_ITER=$(cat $CHECKPOINTS_DIR/latest_checkpoint.txt)
CHECKPOINT_DIR=$CHECKPOINTS_DIR/$CHECKPOINT_ITER

# Convert DCP checkpoint to PyTorch format
python scripts/convert_distcp_to_pt.py $CHECKPOINT_DIR/model $CHECKPOINT_DIR
```

此转换会生成三个文件：

- `model.pt`：包含常规权重和 EMA 权重的完整 checkpoint
- `model_ema_fp32.pt`：仅包含 float32 精度的 EMA 权重
- `model_ema_bf16.pt`：仅包含 bfloat16 精度的 EMA 权重（推荐用于推理）

<a id="32-running-inference"></a>
### 3.2 运行推理

完成 checkpoint 转换后，你可以通过命令行界面使用后训练后的模型运行推理。

#### 单视频生成

```bash
torchrun --nproc_per_node=8 examples/inference.py \
  -i assets/sample_gr00t_dreams_gr1/gr00t_image2world.json \
  -o outputs/gr00t_gr1_sample \
  --checkpoint-path $CHECKPOINT_DIR/model_ema_bf16.pt \
  --experiment predict2_video2world_training_2b_groot_gr1_480
```

> **注意**：对于单个视频，通常 `--nproc_per_node=1` 就足够了。若需要使用批量输入提高吞吐量，可使用 8 块 GPU；请根据你的环境进行调整。

下方展示了一个批量推理输出的可视化示例。

| 提示 | 输入图像 | 生成视频 |
|--------|-------------|----------------|
| 请用右手将红苹果从棕色托盘拿起，放到米色餐垫上。 | <img src="assets/40_Use the right hand to pick up red apple from brown tray to beige placemat..png" width="160"/> | <video width="320" controls autoplay loop muted><source src="assets/40_Use the right hand to pick up red apple from brown tray to beige placemat..mp4" type="video/mp4"></video> |

<a id="4-policy-evaluation-using-cosmos-predict"></a>
## 4. 使用 Cosmos Predict 进行策略评估

最后，我们将下载 [GR00T Eval Dataset](https://huggingface.co/datasets/nvidia/PhysicalAI-Robotics-GR00T-Eval)，然后对其进行预处理以创建批量输入。下面的步骤都在 **cosmos-predict2.5** 仓库根目录下执行。

### 下载 DreamGen Benchmark 数据集

```bash
huggingface-cli download nvidia/EVAL-175 --repo-type dataset --local-dir dream_gen_benchmark
```

### 准备批量输入 json

```bash
python -m scripts.prepare_batch_input_json \
  --dataset_path dream_gen_benchmark/gr1_object/ \
  --save_path output/dream_gen_benchmark/cosmos_predict2_14b_gr1_object/ \
  --output_path dream_gen_benchmark/gr1_object/batch_input.json
```

### 为推理输入预处理批量输入

如果你想根据不同输入和提示生成多个视频，可以使用一个包含批量输入的 JSONL 文件。JSONL 文件应包含一个对象数组，其中每个对象都具有以下字段：

```bash
# Adjust paths based on where you cloned the repositories
cp -r cosmos-cookbook/scripts/examples/predict2.5/gr00t-dreams/gr1_batch_to_jsonl.py cosmos-predict2.5/scripts/
```

请在 **cosmos-predict2.5** 仓库根目录运行以下命令，以便脚本中的路径（`dream_gen_benchmark/gr1_object/batch_input.json` 和 `gr1_batch.jsonl`）能够解析到你已下载并准备好的 eval 数据：

```bash
python scripts/gr1_batch_to_jsonl.py
```

运行上述脚本后，jsonl 文件将具有如下结构：

```jsonl
{"inference_type": "image2world", "name": "000_11_Use_the_right_hand_to_pick_up_green_pepper_from_black_shelf_to_inside_brown_p", "prompt": "Use the right hand to pick up green pepper from black shelf to inside brown paper bag.", "input_path": "11_Use the right hand to pick up green pepper from black shelf to inside brown paper bag..png", "num_output_frames": 93, "resolution": "432,768", "seed": 0, "guidance": 7}
```

### 生成多次视频 rollout

在相同输入下，Cosmos Predict2.5 会生成多个视频 rollout。其中有些结果表现出更强的物理合理性。

```bash
# Adjust paths based on where you cloned the repositories
cp -r cosmos-cookbook/scripts/examples/predict2.5/gr00t-dreams/inference.py cosmos-predict2.5/
cp -r cosmos-cookbook/scripts/examples/predict2.5/gr00t-dreams/config.py cosmos-predict2.5/
```

请在 **cosmos-predict2.5** 仓库根目录运行以下命令，以使用复制过来的 `inference.py` 和 `config.py`：

```bash
torchrun --nproc_per_node=1 inference.py \
  -i dream_gen_benchmark/gr1_object/gr1_batch.jsonl \
  -o outputs/gr1_object_run_ng5 \
  --num-generations 5 \
  --checkpoint-path $CHECKPOINT_DIR/model_ema_bf16.pt \
  --experiment predict2_video2world_training_2b_groot_gr1_480
```

| 第 1 次生成 | 第 2 次生成 | 第 3 次生成 | 第 4 次生成 | 第 5 次生成 |
|---------|---------|---------|---------|---------|
| <video width="320" controls autoplay loop muted><source src="assets/002_28_Use_the_right_hand_to_pick_up_tall_red_glass_from_center_of_tan_table_to_brig_seed0.mp4" type="video/mp4"></video> | <video width="320" controls autoplay loop muted><source src="assets/002_28_Use_the_right_hand_to_pick_up_tall_red_glass_from_center_of_tan_table_to_brig_seed1.mp4" type="video/mp4"></video> | <video width="320" controls autoplay loop muted><source src="assets/002_28_Use_the_right_hand_to_pick_up_tall_red_glass_from_center_of_tan_table_to_brig_seed2.mp4" type="video/mp4"></video> | <video width="320" controls autoplay loop muted><source src="assets/002_28_Use_the_right_hand_to_pick_up_tall_red_glass_from_center_of_tan_table_to_brig_seed3.mp4" type="video/mp4"></video> | <video width="320" controls autoplay loop muted><source src="assets/002_28_Use_the_right_hand_to_pick_up_tall_red_glass_from_center_of_tan_table_to_brig_seed4.mp4" type="video/mp4"></video> |

<a id="5-using-cosmos-reason-as-video-critic-for-rejection-sampling"></a>
## 5. 将 Cosmos Reason 作为视频评论器进行拒绝采样

Cosmos Reason2 能评估视频是否遵循重力、对象恒存性、碰撞动力学和因果关系等基本物理规律。当它与 Cosmos Predict2.5 这样的世界模型搭配使用时，可以通过生成多个候选视频并选择物理上最准确的结果，实现 best-of-N 采样，从而提升生成质量。本配方以**零样本**方式使用 Cosmos Reason 2（不对评论器进行微调）。如果你希望基于物理合理性数据微调评论器，请参阅 [Physical Plausibility Prediction with Cosmos Reason 2](../../post_training/reason2/physical-plausibility-check/post_training.md)。

### 评估标准

每个生成视频都会依据**对物理规律的遵循程度**，按统一的 5 分制接受人工评估：

| **分数** | **说明** | **物理规律遵循度** |
|-----------|-----------------|----------------------|
| **1** | 完全不遵循物理规律 | 完全不合理 |
| **2** | 对物理规律的遵循很差 | 大多不真实 |
| **3** | 对物理规律的遵循程度中等 | 真实与不真实并存 |
| **4** | 对物理规律的遵循良好 | 大多真实 |
| **5** | 完美遵循物理规律 | 完全合理 |

### 零样本推理

???+ code "用于评分物理合理性的提示词"

    ```yaml
    --8<-- "docs/recipes/end2end/gr00t-dreams/assets/video_reward.yaml"
    ```

要运行零样本推理，你需要克隆两个仓库并复制所需文件：

```bash
# Adjust paths based on where you cloned the repositories
cp -r cosmos-cookbook/scripts/examples/predict2.5/gr00t-dreams/inference_videophy2.py cosmos-reason2/examples/gr00t-dreams
cp cosmos-cookbook/docs/recipes/end2end/gr00t-dreams/assets/video_reward.yaml cosmos-reason2/prompts/video_reward.yaml
```

然后在 Cosmos Reason 2 项目根目录下，对生成的视频运行推理：

```bash
uv run examples/gr00t-dreams/inference_videophy2.py \
  --video-dir ./cosmos-predict2.5/outputs/gr1_object_run \
  --output-dir outputs/gr1_object_run_critic
```

上述脚本将为每个生成视频生成一个包含分数和解释的 json 文件。

| **分数** | **视频链接** | **Reason2 解释** |
|-----------|---------------|--------------------|
| 4.0 | <video width="320" controls autoplay loop muted><source src="assets/000_11_Use_the_right_hand_to_pick_up_green_pepper_from_black_shelf_to_inside_brown_p_seed3.mp4" type="video/mp4"></video> | 视频展示了两只机械臂协同工作，将物品装入纸袋中。左臂撑开袋口，右臂把一个绿色甜椒放入袋中。两只机械臂动作平稳且有目的，体现出良好的协同和精度。背景中的人物只是观察流程，没有进行干扰。所有动作都符合真实世界物理规律，没有出现不现实的行为或违反物理定律的现象。物体交互自然，运动流畅且合乎逻辑。 |
| 1.0 | <video width="320" controls autoplay loop muted><source src="assets/000_11_Use_the_right_hand_to_pick_up_green_pepper_from_black_shelf_to_inside_brown_p_seed1.mp4" type="video/mp4"></video> | 机器人手臂未能抓住绿色甜椒，这与其预期功能相矛盾。抓取失败表明控制机器人动作的程序可能存在故障或错误。此外，视频中人物的手始终保持静止，没有与桌上物体发生交互，这与人在类似任务中的典型行为并不一致。 |

<a id="6-computing-metrics"></a>
## 6. 计算指标

<img src="assets/critic_metrics.png" alt="Critic Evaluation Metrics" width="600">

<p>
在 <b>Physical Plausibility</b> 这一标准下，你可以过滤掉得分为 <b>1.0</b> 或 <b>2.0</b> 的合成视频，因为它们对物理规律的遵循很差或完全不遵循。建议保留得分较高的 <b>4.0</b> 或 <b>5.0</b> 视频，这表示其具有良好或完美的物理真实性。
</p>

<a id="limitations-and-considerations"></a>
### 局限性与注意事项

在将这条流水线用于教学或部署时，请记住以下几点：

- **合成数据与真实数据**：生成的轨迹来自学习到的分布；它们可能在物理上合理，但仍会与真实机器人数据存在差异。如果下游策略只在这类合成数据上训练，可能会面临 sim-to-real gap。
- **评论器只是代理指标**：Reason 2 视频评论器只是物理合理性的代理，并非真实标签意义上的裁判。它可能遗漏细粒度的物理现象，也可能对提示词和尺度敏感。Best-of-N 采样能提升质量，但不能保证正确性。
- **领域与规模**：该方法在与 GR1/GR00T 风格操作相近的场景和提示词上表现最佳。若要泛化到差异很大的机器人或任务，可能需要更多数据或额外调优。Best-of-N 和多次 roll out 会增加计算成本，需要按需权衡质量与吞吐量。

<a id="7-conclusion"></a>
## 7. 结论

- **端到端流水线**：准备 GR1 数据 → 在机器人操作数据上后训练 Cosmos Predict 2.5 → 运行推理为每个提示生成多个视频 rollout → 使用 Cosmos Reason 2 为 rollout 打分 → 按物理合理性筛选并计算指标。
- **Best-of-N 拒绝采样**：为每个提示生成多个候选，只保留高分（4.0 或 5.0）视频，用于构建一个经过筛选、具有物理合理性的机器人学习轨迹数据集。
- **可扩展的合成数据**：使用世界基础模型（Predict + Reason）在无需人工标注的情况下，大规模生成并质量过滤合成轨迹。
- **将 Cosmos Reason 2 作为评论器**：使用视频评论器按 1–5 分评估对物理规律（重力、碰撞、因果关系）的遵循程度，只保留合理的 rollout 用于下游训练。

### 引用

如果你使用了此配方或参考了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_gr00t_2026,
  title={Leveraging World Foundation Models for Synthetic Trajectory Generation in Robot Learning},
  author={Apte, Rucha and Jin, Jingyi and Nanda, Saurav},
  organization={NVIDIA},
  year={2026},
  month={March},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/end2end/gr00t-dreams/post-training.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Rucha Apte、Jingyi Jin、Saurav Nanda（2026）。利用世界基础模型为机器人学习生成合成轨迹。载于 *NVIDIA Cosmos Cookbook*。NVIDIA。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/end2end/gr00t-dreams/post-training.html>
