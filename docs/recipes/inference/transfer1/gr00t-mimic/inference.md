# 用于合成操作动作生成的 Isaac GR00T-Mimic

> **Authors:** NVIDIA Isaac Team
>
> **Organization:** NVIDIA

## 概览

| **Model** | **Workload** | **Use Case** |
|-----------|--------------|--------------|
| [Cosmos Transfer 1](https://github.com/nvidia-cosmos/cosmos-transfer1) | Inference | Synthetic manipulation motion generation for humanoid robots |

Isaac GR00T-Mimic 是一个参考工作流，用于基于少量人类示范，为机器人操作任务创建大规模合成运动轨迹。该蓝图基于 **NVIDIA Omniverse™** 和 **Cosmos Transfer 1** 构建，通过生成符合物理规律的合成示范，解决真实世界数据有限的挑战。

## 关键特性

- **数据放大**: 从少量示范集中生成指数级增长的轨迹数量
- **物理准确性**: 利用仿真生成符合物理规律的运动
- **成本效益**: 减少昂贵且耗时的真实世界数据采集
- **泛化能力**: 提供训练稳健机器人学习模型所需的多样性

## 工作原理

1. **人类示范**: 从少量人类操作示范开始
2. **仿真**: 使用 Isaac Sim 和 Omniverse 进行高物理精度的环境仿真
3. **动作合成**: 应用 Cosmos Transfer 1 生成多样化的操作轨迹
4. **策略训练**: 在合成数据集上训练模仿学习模型

## 应用场景

- 人形机器人操作任务
- 物体抓取与放置
- 工具使用与操作
- 灵巧手控制

## 资源

- **[Build Page](https://build.nvidia.com/nvidia/isaac-gr00t-synthetic-manipulation)** - 交互式演示和 API 访问
- **[GitHub Repository](https://github.com/NVIDIA-Omniverse-blueprints/synthetic-manipulation-motion-generation)** - 源代码和文档
- **[Technical Blog](https://developer.nvidia.com/blog/building-a-synthetic-motion-generation-pipeline-for-humanoid-robot-learning/)** - 流程概览与结果
- **[NVIDIA Omniverse](https://www.nvidia.com/en-us/omniverse/)** - 仿真平台
- **[Cosmos Transfer 1](https://github.com/nvidia-cosmos/cosmos-transfer1)** - 多控制视频生成
