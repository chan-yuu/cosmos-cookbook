# Predict 模型评估

> **作者：** [Arslan Ali](https://www.linkedin.com/in/arslan-ali-ph-d-5b314239/)
> **机构：** NVIDIA

本页聚焦于 Predict（生成式视频）模型的评估。它介绍了常用的质量指标，解释每个指标衡量的内容及其工作原理，并提供对应的源码实现与逐步运行说明。

## 关键术语

| 术语 | 定义 |
|------|------------|
| **FID**（Fréchet Inception Distance） | 通过比较从预训练神经网络中提取的特征分布，衡量两组图像之间相似性的指标。 |
| **FVD**（Fréchet Video Distance） | FID 在视频上的扩展，同时衡量空间质量（外观）与时间质量（运动）。 |
| **Fréchet Distance** | 衡量两个概率分布相似性的统计量；在 FID/FVD 中用于量化生成内容与真实内容之间“相距多远”。 |
| **Sampson Error** | 一种几何误差指标，用于衡量匹配关键点到其对应极线的距离；可用于评估多视角一致性。 |
| **TSE**（Temporal Sampson Error） | 在同一相机视角的连续帧之间计算的 Sampson error；用于衡量时间稳定性。 |
| **CSE**（Cross-view Sampson Error） | 在不同相机视角的同步帧之间计算的 Sampson error；用于衡量多视角几何对齐。 |
| **Epipolar Geometry** | 同一三维场景在两个相机视角之间的几何关系；定义了对应点可能出现的位置约束。 |

## 概览：Predict 模型的质量指标

使用以下指标评估生成式视频模型（Predict）：

| 指标 | 衡量内容 | 使用场景 | 更优方向 |
|--------|----------|----------|------------------|
| **FID** | 图像真实感与多样性 | 单帧质量评估 | 越低越好 ↓ |
| **FVD** | 时空视频质量 | 整体视频质量与运动连贯性 | 越低越好 ↓ |
| **TSE** | 时间维度的几何一致性 | 检测视角内闪烁、抖动或漂移 | 越低越好 ↓ |
| **CSE** | 跨视角几何一致性 | 多相机对齐与 3D 一致性 | 越低越好 ↓ |

如需了解基于 VLM 的评估细节，请参阅[Cosmos Reason 作为奖励模型](reason_as_reward.md)以及 [Cosmos Reason Benchmark Example](https://github.com/nvidia-cosmos/cosmos-reason1/blob/main/examples/benchmark/README.md)。

## 指标速查表

使用以下速查表快速解读你的评估分数：

### 视频质量（FID/FVD）

| 评级 | FID 分数 | FVD 分数 | 解释 |
|--------|-----------|-----------|----------------|
| Excellent | < 30 | < 100 | 高质量生成，与 ground truth 非常接近 |
| Good | 30 – 50 | 100 – 200 | 对大多数应用而言质量可接受 |
| Fair | 50 – 100 | 200 – 400 | 可见明显质量差距；建议改进 |
| Poor | > 100 | > 400 | 存在显著质量问题；需要重点关注 |

### 几何一致性（Sampson Error）

| 评级 | TSE/CSE（像素） | 解释 |
|--------|------------------|----------------|
| Excellent | < 1.0 | 极高的几何一致性 |
| Good | 1.0 – 3.0 | 对大多数应用而言可接受 |
| Fair | 3.0 – 5.0 | 可感知到不一致；可能需要改进 |
| Poor | > 5.0 | 存在显著几何误差 |

> **注意**：这些阈值仅为通用参考。可接受范围会因你的具体使用场景、数据集特征和下游应用需求而有所不同。

## 视频质量指标（FID/FVD）

该类指标通过将预测视频与 ground truth 进行比较，以标准化方式评估生成视频的质量。

### 第 1 步：安装指标依赖

```bash
# Install all metrics dependencies
pip install -r scripts/metrics/requirements.txt

# Or install individually:
pip install decord torchmetrics[image] torch-fidelity  # For FID
pip install cd-fvd decord einops scipy                  # For FVD
```

### 第 2 步：计算 FID（Fréchet Inception Distance）

FID 通过比较预训练 Inception 网络中的特征分布，衡量生成帧的质量与多样性。

#### 该指标衡量什么

- 真实图像特征分布与生成图像特征分布之间的距离（越低越好）

#### 该指标如何工作（高层说明）

- 使用 InceptionV3 从真实帧和生成帧中提取 2048 维特征。
- 拟合高斯分布（均值 μ 与协方差 Σ），并计算 Fréchet distance：

```
FID = ||μ_real - μ_gen||² + Tr(Σ_real + Σ_gen - 2√(Σ_real × Σ_gen))
```

#### 示例命令

运行以下命令以计算 FID：

```bash
python scripts/metrics/compute_fid_single_view.py \
    --pred_video_paths "./path/to/predicted/*.mp4" \
    --gt_video_paths "./path/to/ground_truth/*.mp4" \
    --num_frames 57 \
    --output_file fid_results.json
```

#### 参数

- `--pred_video_paths`、`--gt_video_paths`、`--num_frames`、`--output_file`

### 第 3 步：计算 FVD（Fréchet Video Distance）

FVD 将 FID 扩展到视频，评估时空特征，同时捕捉外观与运动。

#### 该指标衡量什么

- 包含时间连贯性与运动动态在内的整体视频质量
- 比纯基于帧的指标更符合用户感知

#### 该指标如何工作（高层说明）

- 为真实视频片段和生成视频片段提取视频特征。
- 类似 FID 地拟合高斯分布并计算 Fréchet distance。

#### 最佳实践

- 保持一致的片段长度、帧率和空间尺寸。
- 调整 `--batch_size` 以平衡吞吐与显存/内存占用。
- 确保预测与 GT 文件列表一一对应，并按字母顺序排序。

#### 示例命令

运行以下命令以计算 FVD：

```bash
python scripts/metrics/compute_fvd_single_view.py \
    --pred_video_paths "./path/to/predicted/*.mp4" \
    --gt_video_paths "./path/to/ground_truth/*.mp4" \
    --num_frames 57 \
    --batch_size 8 \
    --target_size 224 224 \
    --output_file fvd_results.json
```

#### 额外参数

- `--batch_size`：默认值为 8
- `--target_size`：默认值为 224 224

### 输出格式

#### FID 结果（`fid_results.json`）

```json
{
  "FID": 45.23,
  "num_pred_videos": 100,
  "num_gt_videos": 100,
  "num_frames_per_video": 57,
  "total_pred_frames": 5700,
  "total_gt_frames": 5700
}
```

#### FVD 结果（`fvd_results.json`）

```json
{
  "FVD": 123.45,
  "num_pred_videos": 100,
  "num_gt_videos": 100,
  "num_frames_per_video": 57,
  "batch_size": 8,
  "target_size": [224, 224]
}
```

### 重要说明

- **视频数量必须匹配**：这两个指标都要求预测视频与 ground truth 视频数量相等。
- **视频排序**：视频会按字母顺序排序——请确保预测集与 GT 集的命名一致。
- **内存管理**：FID 会将所有帧加载到内存中；FVD 则按 batch 处理——必要时请调整 FVD 的 `--batch_size` 参数。
- **GPU 使用**：若可用，这些脚本会使用 GPU；否则会回退到 CPU。
- **支持格式**：这些脚本通过 `decord` 支持常见视频格式（MP4、AVI、MOV）。

### 示例：完整评估流水线

```bash
# Define video paths
VIDEO_PRED="./generated_videos/*.mp4"
VIDEO_GT="./ground_truth_videos/*.mp4"

# Run FID evaluation
python scripts/metrics/compute_fid_single_view.py \
    --pred_video_paths "$VIDEO_PRED" \
    --gt_video_paths "$VIDEO_GT" \
    --output_file evaluation_fid.json

# Run FVD evaluation
python scripts/metrics/compute_fvd_single_view.py \
    --pred_video_paths "$VIDEO_PRED" \
    --gt_video_paths "$VIDEO_GT" \
    --output_file evaluation_fvd.json
```

## 几何一致性指标（Sampson Error）

这些指标用于评估多视角视频的几何一致性，并诊断 FID/FVD 无法捕捉到的时间不稳定和跨视角错位问题。

该指标具有以下优势：

- 更低的误差意味着更平滑的运动（时间一致性）和更好的多视角几何关系
- 适用于多相机或拼接成 2×3 网格输出的视频

### 第 1 步：为 Sampson 指标配置 Conda 环境

创建并激活专用于 Sampson error 评估的 conda 环境：

```bash
# Navigate to the sampson metrics directory
cd scripts/metrics/geometrical_consistency/sampson/

# Create the conda environment from the yml file
conda env create -f environment.yml

# Activate the environment
conda activate sampson
```

> **注意**：该环境包含 Python 3.10、CUDA 12.4.0 支持，以及所需的视觉相关包（OpenCV、Kornia、PyColmap）。

### 第 2 步：准备多视角视频

请确保视频采用要求的 2×3 网格格式（MP4）：

```
[LEFT    FRONT    RIGHT ]
[REAR_L  REAR_T   REAR_R]
```

### 第 3 步：计算 Sampson Error 指标

运行评估脚本，同时计算 Temporal Sampson Error（TSE）和 Cross-view Sampson Error（CSE）：

```bash
# Single video
python scripts/metrics/geometrical_consistency/sampson/run_cse_tse.py \
    --input path/to/video.mp4 \
    --output ./sampson_results \
    --verbose

# Directory of videos
python scripts/metrics/geometrical_consistency/sampson/run_cse_tse.py \
    --input ./path/to/videos/ \
    --pattern "*.mp4" \
    --output ./sampson_results
```

#### 参数

- `--input`、`--output`、`--pattern`、`--verbose`

### 指标解释

#### Sampson Error（概览）

- 在给定匹配关键点和 fundamental matrix 的条件下，它提供点到极线距离的一阶近似；越低越好。

#### Temporal Sampson Error（TSE）

- 衡量单个视角内连续帧之间的几何一致性；数值越低表示运动越平滑、时间伪影越少。

#### Cross-view Sampson Error（CSE）

- 衡量同步视角之间的几何一致性；数值越低表示多视角对齐越好。

### 输出格式

#### 单视频结果（`cse_tse/{video_id}.json`）

```json
{
  "video_path": "/path/to/video.mp4",
  "clip_id": "video_id",
  "results": {
    "T": {  // Temporal errors
      "front": {
        "mean": 2.345,
        "median": 2.123,
        "frame_values": [...]
      },
      "cross_left": {...},
      "cross_right": {...}
    },
    "C": {  // Cross-view errors
      "front-cross_right": {
        "mean": 3.456,
        "median": 3.234,
        "frame_values": [...]
      },
      "front-cross_left": {...}
    }
  }
}
```

#### 聚合统计（`aggregate_stats.json`）

```json
{
  "num_videos": 10,
  "temporal": {
    "front": {"mean": 2.5, "median": 2.3, "std": 0.8},
    "overall": {"mean": 2.6, "median": 2.4, "std": 0.9}
  },
  "cross_view": {
    "front-cross_right": {...},
    "overall": {...}
  }
}
```

#### 可视化图表（`cse_tse/{video_id}.png`）

生成的图表会显示以下内容：

- **实线**：每个视角的 Temporal Sampson Error（TSE）
- **虚线**：视角对之间的 Cross-view Sampson Error（CSE）
- **Y 轴**：以 √pixels 表示的误差（为便于可视化，截断到 10）
- **X 轴**：帧编号

### 结果解读

**误差范围：**

- **Excellent**（< 1.0 pixels）：几何一致性非常高
- **Good**（1.0 - 3.0 pixels）：对大多数应用而言一致性可接受
- **Fair**（3.0 - 5.0 pixels）：可感知到不一致；可能需要改进
- **Poor**（> 5.0 pixels）：存在显著几何误差

**较高误差意味着什么：**

- **高 TSE**：时间不稳定（单视角内出现闪烁、抖动或漂移）
- **高 CSE**：多视角一致性差（视角错位、几何关系错误）
- **帧级尖峰**：误差突然升高通常意味着问题帧或场景切换

### 示例：完整 Sampson 评估流水线

```bash
# Setup environment
conda activate sampson

# Define video paths
VIDEO_DIR="./generated_multi_view_videos"
OUTPUT_DIR="./evaluation_sampson"

# Run Sampson error evaluation on all generated videos
python scripts/metrics/geometrical_consistency/sampson/run_cse_tse.py \
    --input "$VIDEO_DIR" \
    --pattern "*_gen.mp4" \
    --output "$OUTPUT_DIR" \
    --verbose

# Results will be in:
# - $OUTPUT_DIR/cse_tse/*.json (per-video metrics)
# - $OUTPUT_DIR/cse_tse/*.png (visualization plots)
# - $OUTPUT_DIR/aggregate_stats.json (summary statistics)
```

### 重要说明

关于几何一致性指标，请注意以下事项：

- **视频格式**：视频必须是 2×3 网格格式，包含 6 个相机视角。
- **特征匹配**：这些指标使用 SIFT 特征进行帧/视角间对应匹配。
- **内存使用**：特征提取与匹配需要足够的 RAM。
- **GPU 支持**：如果可用，这些指标会自动使用 GPU 加速以提升处理速度。

---

## 文档信息

**发布日期：** 2025 年 10 月 9 日

### 引用

如果你使用了本内容或引用了本工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_evaluation_predict_2025,
  title={Model Evaluation Predict},
  author={Ali, Arslan},
  year={2025},
  month={October},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/evaluation/evaluation_predict.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用格式：**

> Arslan Ali（2025）。Predict 模型评估。收录于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/core_concepts/evaluation/evaluation_predict.html>

