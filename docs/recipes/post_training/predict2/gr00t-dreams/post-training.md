# Isaac GR00T-Dreams：用于合成轨迹数据生成

> **作者：** NVIDIA Isaac Team
>
> **机构：** NVIDIA

## 概述

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Predict 2](https://github.com/nvidia-cosmos/cosmos-predict2) | 后训练 | 面向人形机器人的合成轨迹数据生成 |

Isaac GR00T-Dreams 利用 **Cosmos Predict 2** 生成合成轨迹数据，用于在新环境中教会人形机器人新的动作。借助 world foundation models，一个小团队就可以创建训练数据，而这些数据原本可能需要数千名示范者才能收集。

## 关键特性

- **可扩展生成**：从极少量人类演示中生成大规模合成轨迹
- **环境泛化**：无需大量重新训练即可适应新环境
- **行为多样性**：覆盖广泛场景和边缘情况
- **成本高效**：大幅减少人工数据采集工作量

## 工作原理

1. **从演示开始**：使用一小组人类演示视频
2. **生成变体**：应用 Cosmos Predict 2 创建带环境变化的合成轨迹
3. **扩展训练数据**：从每个演示生成成千上万种变体
4. **训练策略**：使用合成数据训练鲁棒的机器人控制策略

## 应用

- 人形机器人运动（行走、奔跑、导航）
- 物体操作与交互
- 多地形适应
- 稀有场景和边缘情况覆盖

## 资源

- **[GR00T-Dreams GitHub](https://github.com/nvidia/gr00t-dreams)** - 源代码与文档
- **[Technical Blog](https://developer.nvidia.com/blog/enhance-robot-learning-with-synthetic-trajectory-data-generated-by-world-foundation-models/)** - 深入介绍与结果
- **[NVIDIA Isaac Platform](https://developer.nvidia.com/isaac)** - 机器人开发平台
- **[Cosmos Predict 2](https://github.com/nvidia-cosmos/cosmos-predict2)** - world foundation model
- **[Isaac GR00T](https://developer.nvidia.com/isaac/gr00t)** - 人形机器人 foundation model
