# Transfer 模型评估（ControlNet / Cosmos Transfer）

评估多模态 ControlNet 模型（例如 Cosmos Transfer）时，需要同时关注其对控制信号的保真度以及整体视频质量。

## Predict 指标的适用性

[evaluation_predict.md](evaluation_predict.md) 中记录的所有指标同样适用于 Transfer（ControlNet）模型。请将它们与下文的 ControlNet 专属指标结合使用，以获得更全面的评估。

## 核心指标（控制保真度与技术质量）

### Blur SSIM（结构相似性指数）

该指标会先对预测视频和 ground-truth 视频施加相同的模糊处理，再衡量二者的感知相似性；它对轻微错位更鲁棒，也支持按区域汇报结果。

#### 该指标如何工作

1. 对预测帧和 ground-truth 帧施加相同强度的模糊。
2. 在模糊后的帧上，基于亮度、对比度和结构计算 SSIM。
3. 对帧间逐像素 SSIM 求平均；也可以利用 mask 分别计算前景/背景（FG/BG）。

### Canny-F1 Score

该指标衡量边缘保留的准确度；它将边缘检测视为一个二分类任务，并以 precision/recall 的形式报告 F1。

#### 该指标如何工作

1. 为预测帧和 ground-truth 帧提取 Canny edge map。
2. 将“边缘”定义为正类，将“非边缘”定义为负类。
3. 计算：TP（两者都有边缘）、FP（仅预测中有边缘）、FN（仅 GT 中有边缘）。
4. F1 = 2 × (Precision × Recall) / (Precision + Recall)；同时报告 precision/recall。
5. （可选）使用区域 mask 进行 FG/BG 评估。

### Depth RMSE（均方根误差）

该指标在进行 median scaling 后衡量尺度不变的深度误差，并支持在 log-space 中计算，同时可以屏蔽无效值。

#### 该指标如何工作

1. 使用 Scale-Invariant RMSE（SI-RMSE）以增强对异常值的鲁棒性。
2. Median scaling：ratio = median(GT) / median(pred)
3. 缩放后计算 RMSE：RMSE = sqrt(mean((GT − scaled_pred)²))
4. （可选）在 log-space 中计算；并屏蔽零值/无效深度。

### Seg mIOU（平均交并比）

该指标衡量预测 mask 与 ground-truth mask 之间的分割保真度，并支持灵活的匹配策略。

#### 该指标如何工作

1. 对每个物体/分割区域：IOU = 交集 / 并集
2. 匹配策略：对每个 GT segment 取最大 IOU，或使用 Hungarian 算法做 1 对 1 最优匹配
3. 报告匹配结果的平均 IOU，以及 recall（高于阈值的 GT segment 被检测到的比例）

### Dover Score（视频质量评估）

该指标衡量视频的技术质量，重点关注清晰度、压缩伪影和运动平滑性（而非美学质量）。

#### 该指标如何工作

1. 在完整视频上使用 DOVER（Disentangled Objective Video Quality Evaluator）。
2. 评估清晰度/锐度、压缩伪影、运动平滑性以及整体技术质量。
3. 返回单个质量分数；可以分别对预测视频和 ground-truth 视频计算并进行对比。

