# 面向交通场景的合成数据生成 (SDG)

> **作者：** [Aidan Ladenburg](https://www.linkedin.com/in/aidanladenburg/) • [Adityan Jothi](https://www.linkedin.com/in/adityan-jothi-23a229105)
> **组织：** NVIDIA

| **模型** | **工作负载** | **使用场景** |
|-----------|--------------|--------------|
| [Cosmos Transfer 2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5), [Cosmos Reason 1](https://github.com/nvidia-cosmos/cosmos-reason1), CARLA Simulator | 端到端 | 面向交通场景、结合 VLM 微调的照片级真实感合成数据生成 |

> **先决条件**：此工作流需要特定的 API keys、系统要求和工作流输入。开始之前，请先阅读下面的[先决条件](#prerequisites)部分。

## 概述

本配方演示如何利用 Cosmos 模型为城市交通场景生成照片级真实感的合成数据。该工作流旨在加速智能城市应用中的感知模型和视觉语言模型（VLM）开发。

![主工作流示意图](./assets/main_workflow.png)

---

### 为什么使用 SDG？

在对模型精度要求极高的场景中，基于领域专属数据进行微调至关重要。合成数据生成与增强提供了一种简单且可扩展的方式，可以按你的精确需求收集这类数据。然而，要从仿真器中构建多样化、照片级真实感的训练数据，仍然存在显著挑战：

- **领域鸿沟**：仿真器虽然能提供完美的 ground truth 和可控场景，但其合成外观会造成显著的领域鸿沟，限制了在真实环境部署时，基于仿真数据训练的模型性能。
- **可扩展性限制**：在仿真器中手动构造多样场景需要大量工程投入和计算资源，使得大规模提升数据多样性成本极高。
- **视觉真实感有限**：传统仿真器输出缺乏面向真实世界部署所需的照片级真实感，因此往往还需要额外的后处理或领域自适应技术。

该工作流提供了一套配方，用于：

- 使用 CARLA 模拟自定义交通场景
  - 从仿真中提取 ground truth（RGB、Depth、Segmentation、Normals、2D/3D bounding boxes、事件）
- 使用 COSMOS-Transfer 生成照片级真实感增强结果，缩小 sim-to-real gap
- 通过可定制的增强变量帮助扩展合成数据规模
- 生成用于模型后训练/微调的数据集
  - 利用 SoM-aware 后处理保持跨模态目标对应关系
  - 为 VLM 后训练生成 Q&A 字幕

该配方的输出旨在为后续微调与部署提供一个简洁的交接点。

更多部署相关说明，请参阅 Cosmos Cookbook 的 [Intelligent Transportation Fine-tuning Guide](../../post_training/reason1/intelligent-transportation/post_training.md) 和 [VSS documentation](https://docs.nvidia.com/vss/latest/#) 中的[部署](https://docs.nvidia.com/vss/latest/content/installation-vlms.html#local-models-cosmos-reason1)指南。

---

<a id="prerequisites"></a>
## 先决条件

### 获取 API keys

> ⚠️ **安全警告：** 请将 API keys 存储在环境变量或安全密钥库（例如 HashiCorp Vault、AWS Secrets Manager）中。切勿将 API keys 提交到源代码管理系统，也不要以明文形式分享。

- [NGC API key](https://org.ngc.nvidia.com/setup/api-keys)
  - 配置步骤见[这里](https://docs.nvidia.com/ngc/latest/ngc-user-guide.html#generating-api-key)
- [Hugging Face Token](https://huggingface.co/settings/tokens)：
  - 请确保你的 Hugging Face token 拥有 Cosmos-Transfer2.5 checkpoints 的访问权限
    - 获取一个具有 Read 权限的 [Hugging Face Access Token](https://huggingface.co/settings/tokens)
    - 安装 [Hugging Face CLI](https://huggingface.co/docs/huggingface_hub/en/guides/cli)
    - 运行 `hf auth login` 登录。
    - 阅读并接受 [NVIDIA Open Model License Agreement](https://huggingface.co/nvidia/Cosmos-Predict2.5-2B)
    - 阅读并接受 [Cosmos-Guardrail1 的使用条款](https://huggingface.co/nvidia/Cosmos-Guardrail1)
    - 阅读并接受 [Cosmos-Transfer2.5 的使用条款](https://huggingface.co/nvidia/Cosmos-Transfer2.5-2B)

<a id="workflow-inputs"></a>
### 工作流输入

SDG 工作流需要 3 类独特输入：地图、场景日志和传感器配置。来自 Inverted AI 的这个[仓库](https://github.com/inverted-ai/metropolis/)为每一种输入都提供了少量示例（见快速开始的第 2 步）。关于这些输入的说明以及如何生成你自己的版本，请参阅下列各节。

#### 地图

地图同时包含位置的 3D 模型及其道路定义。地图的道路定义基于 OpenDRIVE 文件。CARLA 提供了一组可用于构建和测试此 SDG 工作流的[预构建地图](https://carla.readthedocs.io/en/latest/catalogue/#maps)。有关地图及其元素的更多细节可见[这里](https://carla.readthedocs.io/en/latest/core_map/)。如果要为真实地点创建数字孪生，可使用带有 CARLA bridge 的 [AVES Reality](https://avesreality.com/) 插件。

#### 场景日志

除了地图外，工作流还需要场景日志。该文件定义了参与者（车辆和行人）列表，以及它们在回放时的精确运动方式，例如碰撞、逆行等。CARLA 提供了一组车辆[资源](https://carla.readthedocs.io/en/latest/catalogue_vehicles/)，可用于仿真。

- 若要生成简单且随机化的交通场景，请参阅 [CARLA 快速开始指南](https://carla.readthedocs.io/en/latest/start_quickstart/#run-a-python-client-example-script)
- 复杂场景可以借助第三方工具创建。其中一个例子是 Mathworks 的 [RoadRunner](https://www.mathworks.com/help/roadrunner/)。也有像 [InvertedAI](https://www.inverted.ai/home) 这样的提供商，可按你的需求生成场景。

场景仿真可以录制并保存为 CARLA 日志文件（自定义二进制文件格式）。随后可回放、查询该日志文件，并用它生成真值数据。有关录制器细节及相关 Python [scripts](https://carla.readthedocs.io/en/latest/adv_recorder/#sample-python-scripts)，请参阅[场景配置](#scenario-configs)部分。

本仓库使用的场景日志可见[这里](https://github.com/inverted-ai/metropolis/tree/master/examples)

<a id="scenario-configs"></a>
#### 场景配置

为了生成真值数据，SDG 工作流需要知道各类 CARLA 传感器的位置及其属性。camera config（`.yaml`）定义了要放置的传感器列表（rgb、depth、seg 等）及其位置、角度和质量。log config（`.json`）提供场景 ID，以及录制时长和起始时间等信息。详情请参阅提供的[示例](https://github.com/inverted-ai/metropolis/tree/master/examples)。

## 系统要求

- Linux，并配有 NVIDIA GPU 和驱动
- Docker Engine 28.0+ 与 Docker Compose v2
- NVIDIA Container Toolkit（用于 GPU 访问）
- Git LFS（Large File Storage）
- 可联网以拉取镜像和模型权重
- 250 GB 存储空间
- 4x RTX GPUs（80+GB Vram）

可选：

- 如果你需要在屏幕上渲染 CARLA，则需要 X11；该堆栈默认使用离屏渲染，但仍默认挂载 X11 以保留灵活性

---

## 工作流使用方式

本配方分为三个阶段：**仿真**、**增强**和**后处理**，完成这些阶段需要 4 个端点（Carla、VLM、LLM、Cosmos Transfer）。本节将介绍在所有端点都已激活前提下的高级用法。若需要了解如何启动这些端点，请参阅[快速开始](#quickstart-docker-compose)；若希望获得基于 docker compose 和 jupyter notebook 的引导式体验，请参阅 [Github](https://github.com/NVIDIA/metropolis-sdg-smart-cities)。

### 阶段 1 - 使用 Carla 仿真生成 GT

该工作流使用开源的 [Carla](https://carla.org/) 仿真器，在多种地图位置上模拟不同类型的交通模式和事故。当前 SDG 版本基于 Carla 0.9.16。此阶段需要 3 类输入信息：用于运行仿真的 Unreal Engine 地图、包含参与者回放信息（车辆/行人运动）的场景日志（`.log`），以及定义摄像头放置位置和记录内容的 sensor config（`.json` / `.yaml`）。为了方便使用，这三个文件的示例都可以在这个[仓库](https://github.com/inverted-ai/metropolis)中找到。关于如何创建自己的场景文件，请参阅[工作流输入](#workflow-inputs)。

<img src="assets/Stage1.png" width="50%">

---

在运行日志仿真之前，你可以先在一个全局配置中自定义一些设置。关于具体变量的更多信息，请参考 [Carla Documentation](https://carla.readthedocs.io/en/latest/python_api/)。

``` json
{
    "host": "localhost",
    "port": 2000,
    "timeout": 360.0,
    "time_factor": 1.0,
    "generate_videos": true,
    "limit_distance": 100.0,
    "area_threshold": 100,
    "class_filter_config": "config/filter_semantic_classes.yaml",
    "ignore_hero": false,
    "move_spectator": false,
    "detect_collisions": true,
    "output_dir": "/path/to/output_dir"
}
```

在指定的全局配置中设置好 host 和 port，并确保 Carla server 正在运行后，你可以像下面这样对单个日志文件运行仿真：

``` bash
python modules/carla-ground-truth-generation/main.py \
            --config /path/to/log_config.json \
            --recorder-filename /path/to/log_file.log \
            --camera-config path/to/camera_config.yaml \
            --wf-config /path/to/global_config.json \
            --output-dir /path/to/output_dir \
            --target-fps 30
```

有关这些文件分别提供什么信息的更多细节，请参见[工作流输入](#workflow-inputs)部分。

生成完成后，你应该会得到一组真值图像：

<img src="./assets/rgb.gif" width="400"><img src="./assets/edges.gif" width="400">

<img src="./assets/seg.gif" width="400"><img src="./assets/depth.gif" width="400">

除了图像之外，仿真还会记录 masks、bbox、collisions 等其他数据。这些数据既可以直接用于微调或训练任务，也可以在工作流下一阶段进一步增强。

### 阶段 2 - 从真值创建增强数据

在第 2 阶段，我们将利用 Carla 生成的真值数据进行增强，以扩展数据集的多样性。这一阶段分为 3 步。首先，使用 Cosmos Reason 1 对输入视频生成字幕，从而得到包含光照、物理事件等属性的详细描述。接着，我们可以借助 LLM 为该提示生成变化版本。目标是在保留场景核心元素的同时，每次只改变少量属性，如一天中的时间或天气。这个步骤可以重复多次，从而为每种变化生成新的增强场景描述。最后，我们可以将这些增强后的提示与真值数据一起传给 Cosmos Transfer 2.5，生成新的增强视频。

此阶段也可以手动编写提示词，但对于大批量增强并不推荐。关于 Cosmos Transfer 的更深入使用说明，请参阅 [CARLA Sim2Real Augmentation Guide](../../inference/transfer2_5/inference-carla-sdg-augmentation/inference.md)

<img src="assets/Stage2.png" width="50%">

---

为了控制 Cosmos Transfer 的生成过程，你可以编写一个简单的配置文件，定义字幕提示词和要使用的增强变量。运行时会从每个列表中随机选择一个变量，用于生成增强后的字幕和视频。下方是一个精简版配置，完整规范请参见 github 上的[sample config](https://github.com/NVIDIA/metropolis-sdg-smart-cities/blob/main/modules/augmentation/configs/config_carla.yaml)。

``` yaml
data:
- inputs:
    rgb: /path/to/Carla-GT/rgb.mp4
    controls:
      edge: /path/to/Carla-GT/edges.mp4
  output:
    video: /path/to/output.mp4
endpoints:
  vlm:
    url: http://localhost:8001/v1
    model: nvidia/cosmos-reason1-7b
  llm:
    url: http://localhost:8002/v1
    model: nvidia/nvidia-nemotron-nano-9b-v2
  cosmos:
    url: http://localhost:8080/
    model: Cosmos-Transfer2.5-2B
video_captioning:
  user_prompt: 'Analyze the traffic intersection surveillance footage and generate a detailed description of the visual elements...'
  variables:
    weather_condition: ['clear_sky', 'overcast', 'snow_falling', 'raining', 'fog']
    lighting_condition: ['sunrise', 'sunset', 'twilight', 'mid_morning', 'afternoon', 'zenith', 'golden_hour', 'blue_hour', 'night']
    road_condition: ['dry', 'snow', 'sand', 'puddles', 'flooding']
cosmos:
  executor_type: gradio
  model_version: ct25
```

设置好配置后，你就可以生成增强视频：

``` bash
python modules/augmentation/modules/cli.py --config /path/to/augmentation_config.yaml
```

**日出与夜晚增强效果对比：**

<img src="./assets/aug.gif" width="400"><img src="./assets/aug2.gif" width="400">

### 阶段 3 - 为后训练任务处理数据

到这里，我们已经成功创建了一个真值数据集，并通过增强提高了其多样性。最后一步是把这些信息打包成可实际用于模型训练或微调的格式。为此，我们将执行两项操作：生成 SOM 叠加结果和 Q&A 对。

SOM（set of marks）是一种结构化标注方法，会为关注目标添加离散标记或标识符。在本场景中，我们会为事故相关的特定车辆添加 bounding boxes 和数字 ID。这些额外标签有助于对 VLM 进行 grounding，从而提升微调质量。

Q&A 对是根据真值数据自动生成的文本提示与回答。它们通过半监督或自监督方式帮助模型从数据集中学习，因此是微调 VLM 的有效机制。

<img src="assets/Stage3.png" width="50%">

---

要叠加真值 bbox 数据，你只需传入增强后的视频以及与之对应的第 1 阶段生成的真值数据。

```bash
python modules/carla-ground-truth-generation/som.py \
      --input-video /path/to/Cosmos-outputs/augmented.mp4 \
      --odvg-dir /path/to/Carla-GT/bbox_odvg \
      --output-video /path/to/SOM.mp4
```

**叠加后的视频：**

<img src="./assets/som.gif" width="400">

利用叠加后的视频，我们可以生成一个用于微调 VLM 的 Q&A 数据集。由于我们知道哪些车辆参与了事故，因此可以基于视频创建大量简单的是/否问题。

``` bash
python modules/postprocess/postprocess_for_vlm.py \
                --carla_folder /path/to/Carla-GT \
                --cosmos_folder /path/to/Cosmos-outputs \
                --output_folder /path/to/output \
                --run_id 1
```

**Q&A 格式：**

```
"id": "events_collision_rgb_som.mp4",​
"video": "events_collision_rgb_som.mp4",​
"conversations": [​
   {"from": "human",​
   "value": "Is there a car collision between vehicles with numeric IDs 968 and 970? Your final ​
   answer should be either Yes or No."},​
   {"from": "gpt",​
   "value": "Yes"}
  ]
```

<a id="quickstart-docker-compose"></a>
## 快速开始（Docker Compose）

1. 克隆仓库

    ```bash
    git clone https://github.com/NVIDIA/metropolis-sdg-smart-cities.git
    cd metropolis-sdg-smart-cities
    ```

1. 下载示例 CARLA 日志

    > **注意：** 示例日志由 Inverted AI 提供。请先查看数据[使用条款](https://github.com/inverted-ai/metropolis/blob/master/LICENSE.md)，以判断它们是否适合你的用途。如果你已有自己的数据，可以跳过此步骤并将其放入 `./data/examples/`。

    ```bash
    git clone https://github.com/inverted-ai/metropolis.git
    mv ./metropolis/examples ./data/examples
    ```

1. 设置部署配置。

    你需要提供 `NGC_API_KEY`（需具备[从 build.nvidia 拉取镜像](https://build.nvidia.com/settings/api-keys)的权限）以及对[先决条件](#prerequisites)中所述 checkpoints 具有访问权限的 Hugging Face Token。其他参数是可选项，用于配置各个 NIM/service 使用的 GPU ID，以及启动这些 NIM 的端口。默认配置假设部署在至少配备 4x RTX 6000 Pro 或同等级 GPU 的同构系统上。

    ```bash
    cd deploy/compose
    cp env.example env
    # Edit values for NGC_API_KEY, HF_TOKEN, GPU IDs, ports, etc.
    ```

1. 部署整个堆栈。

    部署脚本在启动容器之前会自动执行先决条件检查：

    - **GPU 可用性**：验证是否检测到并可访问 NVIDIA GPUs
    - **NVIDIA Container Toolkit**：确认容器中的 GPU 访问配置正确
    - **端口可用性**：检查所需端口（8001、8002、8080、8888、2000-2002）是否已被占用
    - **Docker 和 Docker Compose**：验证所需工具已安装且 Docker daemon 正在运行

    如果任何关键检查失败，脚本会给出清晰的错误信息并退出。请先解决相关问题，再重新尝试部署。

    有两种主要的部署方式：

    - **同构部署：** 此模式会在单台机器上启动所有 NIM 服务（VLM、LLM、Cosmos-Transfer）和 Workbench（默认模式，无需额外参数）。建议用于至少拥有 4 张合适 GPU（支持 RTX 且显存 80+ GB）的系统。只需运行 `./deploy.sh` 即可在本地启动整个堆栈。

    ```bash
    # On the target machine
    ./deploy.sh

    # This spins up the Cosmos-Reason1, Nemotron NIMs, Cosmos-Transfer2.5 Gradio Server, CARLA Server, and the Jupyter notebook, which users can follow to generate photo-realistic synthetic data for VLMs.
    # By default these are the ports where all of the services get deployed to.
    # Workbench → http://<host>:8888
    # NIMs: VLM http://<host>:8001, LLM http://<host>:8002, Cosmos-Transfer http://<host>:8080
    ```

    > **注意：** 首次运行时，你可能会看到诸如“pull access denied for `smartcity-sdg-workbench`”或 Transfer Gradio 容器的警告。这是预期且无害的——`deploy.sh` 会在初始设置期间本地构建所需镜像。

    - **异构部署：** 此模式允许你在一台机器上运行 NIM 堆栈（VLM、LLM、Cosmos-Transfer），在另一台机器上运行 Workbench（带 CARLA），分别使用 `nim` 和 `workbench` 参数。这适合希望将资源负载分散到多台主机的场景。你需要在 Workbench 节点上设置 `NIM_HOST` 环境变量，使其指向 NIM 节点。

    NIM 堆栈需要一台配备 3 张 80+ GB VRAM GPU（Ampere 或更新架构）的机器，以通过下述命令启动 3 个推理端点：

    ```bash
    ./deploy.sh nim
    # Note the printed NIM_HOST and use it on the workbench node.
    ```

    当 NIM 堆栈启动后，再启动 CARLA server 和 notebook/workbench 堆栈；后者至少需要 1 张兼容 RTX 的 GPU（L40/RTX 6000 Pro 或同等级）并使用以下命令：

    ```bash
    # On the second machine, ensure steps 1-3 are complete to have the repository and configuration ready before this step.
    # The deployment script sources `deploy/compose/env`, where `NIM_HOST` defaults to `localhost`. This will override any previously exported `NIM_HOST`. Before running `./deploy.sh workbench`, edit `deploy/compose/env` and set `NIM_HOST=<ip_of_nim_node>`. The script will prompt you to confirm the detected value.
    cd deploy/compose
    ./deploy.sh workbench
    ```

    请根据你的可用硬件和工作流需求选择最合适的方案。

1. 验证部署并开始使用系统

    **注意：** 首次部署时，NIM 需要数分钟下载模型 checkpoints 并完成初始化。请等待几分钟后再访问服务。

    **检查 NIM 健康检查端点：**

    ```bash
    # If using heterogeneous deployment, set NIM_HOST to the NIM node IP first:
    # export NIM_HOST=<ip_of_nim_node>
    HOST=${NIM_HOST:-localhost}
    curl http://$HOST:8001/v1/health/ready  # VLM should return "Service is live."
    curl http://$HOST:8002/v1/health/ready  # LLM should return "Service is live."
    ```

    - Cosmos-Transfer2.5 Gradio service:
      - notebook 通过 Gradio client 与 Gradio server 通信。在浏览器中打开 `http://localhost:8080`（或异构部署下的 `http://$NIM_HOST:8080`）是可选的，主要用于确认服务已经启动。

    - 打开 Workbench（Jupyter）：
      - 访问 `http://localhost:8888`（若为异构部署，则访问 `http://<WORKBENCH_HOST>:8888`）。
      - 打开 notebook `notebooks/carla_synthetic_data_generation.ipynb`。这是一个自引导式流程，使用已部署服务覆盖全部三个阶段：
        - 阶段 1：CARLA 真值生成
        - 阶段 2：COSMOS 照片级真实感增强
        - 阶段 3：面向 VLM 训练的 SoM 对齐后处理

1. 清理（完成后）

    要停止并移除所有容器：

    ```bash
    cd deploy/compose
    ./deploy.sh cleanup
    ```

    这会停止并移除 NIM 和 Workbench 两个堆栈中的所有容器。若采用异构部署，请在两个节点（NIM 节点和 Workbench 节点）上都运行此命令，以彻底清理所有容器。

## 资源

### 相关 Cookbook 配方

- **[Cosmos Transfer 2.5 用于仿真器视频的 Sim2Real](../../inference/transfer2_5/inference-carla-sdg-augmentation/inference.md)** - 深入了解 CARLA 仿真驾驶数据的增强技术
- **[Intelligent Transportation Fine-tuning](../../post_training/reason1/intelligent-transportation/post_training.md)** - 关于如何在生成的合成数据上微调 VLM 的指南
- **[CARLA Simulator](https://carla.org/)** - 官方 CARLA 文档与教程

### 部署与集成

- **[SDG for Smart Cities GitHub](https://github.com/NVIDIA/metropolis-sdg-smart-cities)** - 包含 Docker Compose、配置文件和 Jupyter notebooks 的完整部署堆栈
- **[VSS Documentation](https://docs.nvidia.com/vss/latest/#)** - 在 VSS 上部署经过微调的模型，可参见 [Cosmos Reason1 on VSS](https://docs.nvidia.com/vss/latest/content/installation-vlms.html#local-models-cosmos-reason1)

### 使用的模型

- **[Cosmos Transfer 2.5](https://research.nvidia.com/labs/dir/cosmos-transfer2.5/)** - 面向照片级真实感增强的多控制视频生成
- **[Cosmos Reason 1](https://research.nvidia.com/labs/dir/cosmos-reason1/)** - 用于视频字幕生成的视觉语言模型
- **[Nemotron](https://developer.nvidia.com/nemotron)** - 用于提示增强与变体生成的 LLM

---

## 文档信息

**发布日期：** 2025 年 11 月 26 日

### 引用

如果你使用了此配方或参考了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_synthetic_data_generation_2025,
  title={Synthetic Data Generation (SDG) for Traffic Scenarios},
  author={Ladenburg, Aidan and Jothi, Adityan},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/end2end/smart_city_sdg/workflow_e2e.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Aidan Ladenburg、Adityan Jothi（2025）。面向交通场景的合成数据生成 (SDG)。载于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/end2end/smart_city_sdg/workflow_e2e.html>
