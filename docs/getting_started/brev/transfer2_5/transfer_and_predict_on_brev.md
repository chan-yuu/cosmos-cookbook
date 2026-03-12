# 在 Brev 上开始使用 Transfer2.5 和 Predict2.5

> **作者：** [Carlos Casanova](https://www.linkedin.com/in/carloscasanova/)
> **组织：** NVIDIA

## 探索 Brev

NVIDIA Brev 是一个非常适合实验 Cosmos 的平台。按照以下步骤即可开始：

1. 在 [https://brev.nvidia.com](https://brev.nvidia.com) 创建账户。
2. 按照 [https://docs.nvidia.com/brev/latest/brev-cli.html](https://docs.nvidia.com/brev/latest/brev-cli.html) 安装 CLI。
3. 参考 [Brev Quickstart](https://docs.nvidia.com/brev/latest/quick-start.html) 熟悉平台。Brev 页面中也链接了 Brev 文档。

虽然某些工作流在更低规格的 GPU 上也能运行，但对于 Cosmos，推荐使用具备 80GB VRAM 的 GPU。另外，Transfer 2.5 AV Multiview 模型需要 8 张或更多 GPU 的实例。

## 快速捷径：Launchables

[Launchables](https://docs.nvidia.com/brev/latest/launchables.html) 是一种便捷方式，可将硬件和软件环境打包为易于分享的链接。一旦你把 Cosmos 环境调试好，Launchable 就是节省时间并与他人共享配置的最方便方式。

本节将带你为 Transfer2.5 构建一个 Launchable。Predict2.5 的设置过程与下面步骤几乎完全相同。请参考 [Predict2.5 setup guide](https://github.com/nvidia-cosmos/cosmos-predict2.5/blob/main/docs/setup.md)，并相应调整设置脚本。你也可以同时配置这两个模型。

> **注意**：Cosmos 和 Brev 都在不断演进。随着 Brev 的变化，你在下面的步骤中可能会看到一些细微的 UI 或其他差异。

1. 在 Brev 网站中找到 **Launchable** 部分。

   ![Launchables Menu](images/brev01-launchable-menu.png)

2. 点击 **Create Launchable** 按钮。

   ![Create Launchable Button](images/brev02-create-launchable-button.png)

3. 输入 Cosmos Transfer URL： [https://github.com/nvidia-cosmos/cosmos-transfer2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5)

   ![Cosmos Transfer URL](images/brev03-create-launchable-step1.png)

4. 添加一个设置脚本。Brev 会在克隆仓库后运行它。该脚本应遵循 [Cosmos Transfer2.5 repo](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/setup.md) 中的设置说明。在本示例中，我们使用本指南后面提供的 [sample script](#sample-setup-script)，它会构建 Transfer2.5 Docker 镜像，并在你的 Brev 环境主目录中创建另一个脚本来启动容器。

   ![Add setup script](images/brev04-create-launchable-step2.png)

5. 如果你不需要 Jupyter，可以将其移除。如果你计划搭建自定义服务器，也可以在 Brev 上开放其他端口。

   ![Add ports](images/brev05-create-launchable-step3.png)

   > Predict2.5 的设置与上述步骤几乎完全相同。请参考 [Predict2.5 setup guide](https://github.com/nvidia-cosmos/cosmos-predict2.5/blob/main/docs/setup.md)，并相应调整设置脚本。想一次性把两个都配好？完全可以，尽管去做吧。

6. 选择所需的算力级别。下图演示了如何筛选 8 张及以上 GPU，以运行 Transfer 2.5 AV Multiview 模型。

   ![Choose compute](images/brev06-create-launchable-step4.png)

7. 为你的 Launchable 命名并配置访问权限。

   ![Name and configure access](images/brev07-create-launchable-step5.png)

   现在你已经可以部署了！注意 **View All Options** 链接，它允许你更改算力配置。

   ![Ready to deploy](images/brev08-launchable-ready-to-deploy.png)

8. 部署完成后，访问实例页面以查看一些有用的连接示例。注意 **Delete** 按钮，它允许你在使用完后删除实例。你也可以通过 `brev delete` CLI 命令完成此操作。支持暂停和恢复的实例也可以在此页面停止。

   ![Instance page](images/brev09-instance-page.png)

9. 连接到实例。本示例运行生成的 `run_transfer2.5_docker.sh` 脚本来启动容器。出现提示符后，运行 `hf auth login` 以启用 checkpoint 下载。没有这些 checkpoint，Transfer2.5 将无法工作。

   ![Docker prompt](images/brev10-docker-prompt.png)

   > Docker entrypoint 会拉取依赖项，并且由于 Python 虚拟环境（venv）文件夹与容器共享，后续运行时这些依赖通常已经安装完成。

<a id="sample-setup-script"></a>
### 示例设置脚本

下面的示例设置脚本会构建一个 Transfer2.5 Docker 镜像，并在你的 Brev 环境主目录中创建另一个脚本用于启动容器。进入容器后，请运行 `hf auth login` 命令以启用 checkpoint 下载。更多信息请参阅 [Transfer2.5 Downloading Checkpoints](https://github.com/nvidia-cosmos/cosmos-transfer2.5/blob/main/docs/setup.md#downloading-checkpoints) 章节。

```bash
#!/bin/bash

# Detect the ultimate Brev user. We will run the script as them.
if id "nvidia" &>/dev/null; then
  RUN_USER="nvidia"
elif id "shadeform" &>/dev/null; then
  RUN_USER="shadeform"
elif id "ubuntu" &>/dev/null; then
  RUN_USER="ubuntu"
else
  RUN_USER="root"
fi

sudo -u $RUN_USER bash -lc '
# Install required packages
sudo apt-get update && sudo apt-get install -y git-lfs bc

# Move into $HOME/cosmos-transfer2.5
cd $HOME/cosmos-transfer2.5

# Initialize git-lfs and pull LFS files to ensure complete clone
git lfs install
git lfs pull

# Build the Cosmos Transfer 2.5 docker image
docker build --ulimit nofile=131071:131071 -f Dockerfile . -t transfer2.5

# Create folders to share HuggingFace files and .venv with the container
HF_HOME=$HOME/.cache/huggingface
VENV_DIR=$HOME/.venv_transfer2.5
mkdir -p $HF_HOME
mkdir -p $VENV_DIR

# Find out number of GPUs and CPUs
NUM_GPUS=$(nvidia-smi --query-gpu=name --format=csv,noheader | wc -l)
NUM_CPUS=$(nproc)

# Set OMP_NUM_THREADS to FLOOR(NUM_CPUS/NUM_GPUS)
OMP_NUM_THREADS=$(echo "scale=0; $NUM_CPUS/$NUM_GPUS" | bc)

# Create script to run the container
VOL="-v $HF_HOME:/root/.cache/huggingface -v $HOME/cosmos-transfer2.5:/workspace -v $VENV_DIR:/workspace/.venv"
ENV="-e HF_HOME=/root/.cache/huggingface -e OMP_NUM_THREADS=$OMP_NUM_THREADS -e NUM_GPUS=$NUM_GPUS"
RUN_CMD="docker run -it --rm --ipc=host --name transfer2.5 $VOL $ENV transfer2.5"
echo "$RUN_CMD" > $HOME/run_transfer2.5_docker.sh
chmod +x $HOME/run_transfer2.5_docker.sh
'
```

## 说明与提示

- 我们建议使用具有 80GB 及以上 VRAM 的 GPU。
- 我们建议使用存储容量为 2TB 或以上的实例。低于 2TB 时，你可能会遇到空间不足的问题。
- 使用完成后，别忘了关闭（即删除）你的实例。
- 截至 2025 年 11 月，大多数适合运行 Transfer 2.5 和 Predict 2.5 的实例并不支持暂停与恢复（启动/停止）功能。
- 在评估实例类型时，请注意 Brev 给出的部署时间预估（例如 "Ready in 7minutes"）。
- 部署有时会失败，而且在尝试新的提供商时，驱动版本可能也不符合预期。因此，建议预留 3 倍于预估就绪时间的缓冲，你会轻松很多 😀
- 你喜欢的云服务提供商并不总是随时可用。
- 你可以更改 Launchable 的算力配置。以下是一些这样做的原因：
  <ul>
    <li>☁️ 首选的云服务提供商当前不可用。</li>
    <li>💰 你想通过不同的配置节省成本。</li>
    <li>🏎️ 你想尝试更高规格。</li>
  </ul>
