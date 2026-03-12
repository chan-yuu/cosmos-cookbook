# 将 Time Series K-Means 应用于嵌入向量的数据集视频聚类

> **作者：** [Petr Khrapchenkov](https://jp.linkedin.com/in/petr-khrapchenkov)
> **组织：** [AI Robot Association (AIRoA)](https://www.airoa.org/)

| **模型** | **工作负载** | **使用场景** |
|-----------|--------------|--------------|
| [Cosmos Curator](https://github.com/nvidia-cosmos/cosmos-curate)  | 数据整理 | 基于嵌入向量轨迹的 Time Series K-Means 视频聚类 |

## 概述

本配方展示了一种最小化、可复现的方法，用于**分析和可视化预先计算好的嵌入**（例如由 [Cosmos Curator](https://github.com/nvidia-cosmos/cosmos-curate) 生成的嵌入）。

你将会：

1. 加载示例数据
2. 通过 **2D UMAP**（余弦距离）对数据进行插值并降维
3. 使用 **TimeSeriesKMeans (softDTW)**（tslearn）对**整段 episode 轨迹**进行聚类
4. 使用 Matplotlib 可视化结果

> **为什么要做轨迹聚类？**
>
> 点级聚类往往会混淆时间行为。轨迹聚类会按照嵌入随时间变化的方式对*完整 episode* 进行分组，这对于机器人行为分析和数据 QA 往往更有意义。

### 背景动机

很多人采用 Cosmos Curator 是因为它针对视频字幕生成做了性能优化，但你同样可以在 Cosmos Curator 生成的视频片段嵌入之上实现更高级的聚类算法。在 [AIRoA](https://www.airoa.org/) ，我们一直在使用 Cosmos Curator 生成的嵌入运行 Time Series K-Means，以帮助对机器人执行示例任务时的行为记录进行分类和聚类。

### 概览

本配方演示了可应用于 Cosmos Curator 数据整理输出结果的高级聚类技术。你将学习如何获取一组来自多个输入视频的嵌入向量，将其转换为格式正确、结构清晰的数据集，然后运行 Time Series K-Means 聚类算法，以识别并区分相似/不同的输入视频。

本指南聚焦于一个最小化的演示工作流，并提供示例数据集和 Jupyter Notebook 实现。

### 核心思想

许多人都熟悉 [K-Means clustering algorithm](https://en.wikipedia.org/wiki/K-means_clustering) 对数据*点*自动聚类的能力；而只要采用适合时间序列的距离度量（如 [Dynamic Time Warping (DTW)](https://en.wikipedia.org/wiki/Dynamic_time_warping)，更具体地说是其可微分变体 [Soft-DTW](https://arxiv.org/abs/1703.01541)），就可以把 K-Means 方法应用到完整的多维时间序列上：这与 Cosmos Curator 处理长视频时生成的数据结构完全一致——它会把长输入视频切成较短片段，并为每个片段生成一个嵌入向量（从而为每个长视频产生一条嵌入向量“时间序列”）。

### 文件

1. [JSON 示例数据文件](https://github.com/nvidia-cosmos/cosmos-cookbook/releases/download/assets/embedding_analysis_trajectories.json)
2. Jupyter Notebook 实现

下面的说明基于如下 `uv` + jupyter notebook 环境进行了测试。

```shell
uv init --python 3.12

uv add scikit-learn umap-learn scipy matplotlib jupyterlab tslearn

uv run jupyter lab
```

### 配方步骤

1. 了解正确的数据格式 / 结构
2. 降维与插值
3. 运行 Time Series K-Means 算法
4. 检查结果

## 执行分析

### 1 - 输入数据格式 / 结构

当你在目标数据集上运行完 Cosmos Curator 后，就可以把它为各个子片段生成的嵌入向量整理成多种形式和结构用于分析。如果这些片段是通过固定步长的 curator 选项处理的，而且窗口相对较短，效果会更好。这样 Cosmos Curator 生成的嵌入序列会足够密集。5 秒大概是一个合理的起点；如果数据中包含更快的运动，可以尝试减小窗口大小；如果运动较慢，则可以适当增大。

为了简化这个演示，我们假设你可以将感兴趣的每个片段嵌入向量收集成如下列表：

```py
# Sample of what the data structure might look like
# all_video_data = [
#  [ #all clips for video 1
#   [1.0, 2.0, 3.0,...], # embedding vector for clip1
#   [1.0, 2.0, 3.0,...], # embedding vector for clip2
#   ...
# ],
# [ #all clips for video 2
#   [1.0, 2.0, 3.0,...], # embedding vector for clip1
#   [1.0, 2.0, 3.0,...], # embedding vector for clip2
#   ...
# ]
#  ...
# ]
```

请注意，每个视频的片段数量可能不同，因此输入时不能直接使用 numpy ndarray。

为了便于跟随示例操作，我们在这里提供了一个来自机器人项目的真实示例数据集，你可以下载并配合我们的 Jupyter Notebook 一起使用。

```py
import json
trajectories = json.load(open("embedding_analysis_trajectories.json", "r"))
```

### 2 - 降维与插值

当前的数据还不能直接用于 Time Series K-Means：我们需要先解决两个问题。

首先，根据输入视频的长度和内容，Cosmos Curator 可能会为每个视频生成不同数量的片段。由于我们要使用的 Time Series K-Means 要求输入时间序列长度一致，因此需要把每个视频已有的嵌入向量时间序列插值到一个所有视频共享的固定长度。

其次，由于嵌入向量位于高维空间中，出于计算效率考虑，我们应在运行 Time Series K-Means 之前先进行降维。

插值函数如下：

```py
import numpy as np

def subdivide_trajectory(trajectory: np.ndarray, n_points: int) -> np.ndarray:
    traj = np.asarray(trajectory, dtype=float)
    t_len, n_features = traj.shape
    if t_len == 1:
        return np.repeat(traj, repeats=n_points, axis=0)

    x_old = np.linspace(0.0, 1.0, t_len)
    x_new = np.linspace(0.0, 1.0, int(n_points))

    out = np.empty((int(n_points), n_features), dtype=float)
    for feat_idx in range(n_features):
        out[:, feat_idx] = np.interp(x_new, x_old, traj[:, feat_idx])
    return out

interpolated_trajectories = np.asarray(
    [subdivide_trajectory(np.asarray(traj), 6) for traj in trajectories]
)
```

降维：

```py
n_traj, t_len, dim = interpolated_trajectories.shape
flat = interpolated_trajectories.reshape(n_traj * t_len, dim)

from umap import UMAP
RUN_SEED = 353550416
flat_2d = UMAP(
    n_components=2,
    random_state=RUN_SEED,
    n_neighbors=15,
    min_dist=0.1,
    metric="cosine",
).fit_transform(flat)
```

### 3 - 运行 Time Series K-Means 算法

现在数据已经准备就绪，我们将使用 [tslearn](https://tslearn.readthedocs.io/) 中的 [Time Series K-Means algorithm](https://tslearn.readthedocs.io/en/stable/gen_modules/clustering/tslearn.clustering.TimeSeriesKMeans.html)，并采用前文提到的 [Soft-DTW](https://arxiv.org/abs/1703.01541) 度量。

```py
from tslearn.clustering import TimeSeriesKMeans

trajectories_2d = flat_2d.reshape(n_traj, t_len, 2)

RUN_SEED = 353550416
model = TimeSeriesKMeans(
    n_clusters=3,
    metric="softdtw",
    max_iter=50,
    random_state=RUN_SEED,
    verbose=False,
)
traj_labels = model.fit_predict(trajectories_2d)
centers = np.asarray(model.cluster_centers_)
```

> 为什么使用 Soft-DTW？
>
> 将 K-Means 技术应用于时间序列数据时，首要挑战在于选择合适的距离度量。
>
> 如果两条时间序列在时间轴上存在偏移或形变，逐点欧氏距离可能会失效。> [Dynamic Time Warping](https://en.wikipedia.org/wiki/Dynamic_time_warping)（DTW）正是为了解决这个问题而提出的。
> ![](assets/euclidean.png)
>
> ![](assets/soft-dtw.png)
>
> [Soft-DTW](https://arxiv.org/abs/1703.01541) 是常规 Dynamic Time Warping 的可微分变体。

### 4 - 检查结果

最后，让我们来看一下结果。首先，我们将使用 [matplotlib](https://matplotlib.org/) 对聚类进行可视化：

```py
import matplotlib.pyplot as plt

flat_points = trajectories_2d.reshape(-1, 2)
flat_labels = np.repeat(traj_labels, t_len)

cmap = plt.get_cmap("tab10")
unique = sorted({int(x) for x in np.asarray(flat_labels).tolist()})
color_map = {cid: cmap(i % cmap.N) for i, cid in enumerate(unique)}

plt.figure(figsize=(10, 8))
for cid in sorted(color_map.keys()):
    m = flat_labels == cid
    plt.scatter(flat_points[m, 0], flat_points[m, 1], s=6, alpha=0.7, color=color_map[cid], label=str(cid))
plt.title("All points (2D) colored by trajectory cluster")
plt.xlabel("Component 1")
plt.ylabel("Component 2")
plt.legend(title="trajectory_cluster", loc="best", markerscale=2, fontsize=9)
plt.show()

# Plot 2: trajectories + barycenters
plt.figure(figsize=(10, 8))
for i in range(n_traj):
    cid = int(traj_labels[i])
    plt.plot(trajectories_2d[i, :, 0], trajectories_2d[i, :, 1], linewidth=1.0, alpha=0.35, color=color_map.get(cid, (0,0,0,0.35)))
for cid in range(centers.shape[0]):
    plt.plot(centers[cid, :, 0], centers[cid, :, 1], linewidth=4.0, alpha=1.0, color=color_map.get(int(cid), (0,0,0,1.0)), label=f"cluster {cid} barycenter")
plt.title("Trajectory clusters + barycenters")
plt.xlabel("Component 1")
plt.ylabel("Component 2")
plt.legend(loc="best", fontsize=9)
plt.show()
```

![](assets/all_points.png)

![](assets/clusters.png)

如果你想进一步用定量指标评估聚类质量，还可以使用多种方法，例如 [silhouette scores](https://en.wikipedia.org/wiki/Silhouette_\(clustering\))（[scikit-learn implementation](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.silhouette_score.html)）或 [Davie-Bouldin scores](https://en.wikipedia.org/wiki/Davies%E2%80%93Bouldin_index)（[scikit-learn implementation](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.davies_bouldin_score.html)），帮助你评估效果并选择超参数（如聚类数目）。

```py
from sklearn.metrics import silhouette_score, davies_bouldin_score

traj_means = trajectories_2d.mean(axis=1)  # (n_traj, 2)

print("silhouette_score:", float(silhouette_score(traj_means, traj_labels)))
print("davies_bouldin_score:", float(davies_bouldin_score(traj_means, traj_labels)))
```

示例 silhouette 与 Davies-Bouldin 分数：

```
silhouette_score: 0.6653785109519958
davies_bouldin_score: 0.42510679230192855
```

---

## 文档信息

**发布日期：** 2026 年 1 月 6 日

### 引用

如果你使用了此配方或参考了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_dataset_video_clustering_2026,
  title={Dataset Video Clustering with Time Series K-Means Applied to Embedding Vectors},
  author={Khrapchenkov, Petr},
  organization={AI Robot Association (AIRoA)},
  year={2026},
  month={January},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/data_curation/embedding_analysis/embedding_analysis.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Petr Khrapchenkov（2026）。将 Time Series K-Means 应用于嵌入向量的数据集视频聚类。载于 *NVIDIA Cosmos Cookbook*。AI Robot Association (AIRoA)。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/data_curation/embedding_analysis/embedding_analysis.html>
