# 在 Brev 上开始使用 Cosmos Reason 2：推理与后训练
>
> **作者：** [Saurav Nanda](https://www.linkedin.com/in/sauravnanda/)
> **组织：** NVIDIA

本指南将带你在 [Brev](https://brev.dev) 的 H100 GPU 实例上部署 NVIDIA [Cosmos Reason 2](https://huggingface.co/nvidia/Cosmos-Reason2-8B)，用于推理和后训练工作流。Brev 提供按需的云端 GPU 和预配置环境，让你能够轻松开始使用 Cosmos 模型。

## 概览

[Brev.dev](https://brev.dev) 是一个云 GPU 平台，可即时访问 H100 等高性能 GPU。本指南将帮助你：

1. 创建一台配备 H100 GPU 的 Brev 实例
2. 为 Cosmos Reason 2 配置环境
3. 运行 Reason 2 模型推理
4. 在自定义数据集上执行后训练（SFT）

## 前置条件

- 一个 Brev 账户（[在此注册](https://brev.dev)）
- 按照 [https://docs.nvidia.com/brev/latest/brev-cli.html](https://docs.nvidia.com/brev/latest/brev-cli.html) 安装 CLI
- 参考快速入门以熟悉该平台：[https://docs.nvidia.com/brev/latest/quick-start.html](https://docs.nvidia.com/brev/latest/quick-start.html)。Brev 页面中也提供了便捷的文档链接。
- 一个 Hugging Face 账户，并已获得 [Cosmos Reason 2](https://huggingface.co/nvidia/Cosmos-Reason2-8B) 的访问权限

## 快速捷径：Launchables

[Launchables](https://docs.nvidia.com/brev/latest/launchables.html) 是一种便捷方式，可将硬件和软件环境打包为易于分享的链接。一旦你把 Cosmos 环境调试好，Launchable 就是节省时间并与他人共享配置的最方便方式。

> **注意：** Cosmos 和 Brev 都在不断演进。随着时间推移，你在下面的步骤中可能会遇到一些细微的 UI 或其他差异。

## 第 1 步：创建一个 Brev Launchable

1. 登录你的 [Brev 账户](https://brev.dev)
2. 在 Brev 网站中找到 Launchable 部分。
![Launchables Menu](./images/brev-01-launchable-menu.png)
3. 点击 **Create Launchable** 按钮。
![Create Launchable Button](./images/brev-02-create-launchable-button.png)
4. 输入 Cosmos Reason 的 [GitHub URL](https://github.com/nvidia-cosmos/cosmos-reason2) ![https://github.com/nvidia-cosmos/cosmos-reason2](./images/brev-03-chose-repo.png)

5. 为 Cosmos Reason 添加一个设置脚本。示例可参考 [sample setup script](./setup_script.sh)。
![Setup Script](./images/brev-06-startup-script.png)

6. 如果你不需要 Jupyter，可以将其移除。如果你计划搭建其他自定义服务器，请告诉 Brev 需要开放哪些端口（如果有）。 ![Jupyter](./images/brev-05-chosejupyter.png)

7. 选择一台具有 80GB VRAM 的 H100 GPU 实例。
![H100 Instance](./images/brev04-choose-compute.png)

8. 为你的 Launchable 命名并配置访问权限。（通常需要 2-3 分钟）。 ![create](./images/brev-07-create-launchable.png)

## 第 2 步：部署并连接到你的实例

1. 在 launchables 列表中，点击 "Deploy Now" 按钮。
![Deploy Now](./images/brev-08-deploy-0.png)

2. 接着在详情页点击 "Deploy Launchable" 按钮。
![Deploy Launchable](./images/brev-08-deploy-1.png)

3. 点击 "Go to Instance Page" 按钮。
![Go to Instance Page](./images/brev-08-deploy-launchable.png)

4. 当实例准备就绪后，Brev 会提供 SSH 连接信息。
![instance](./images/brev-09-access-or-stop.png)

### 选项 1：打开 Jupyter Notebook

![Notebook](./images/brev-10-notebook.png)

### 选项 2：从你的 Brev 控制台复制 SSH 命令

```bash
brev login --token <YOUR_TOKEN>
```

在本地打开一个终端

```bash
brev shell sample-reason2-bdb1b0
```

或者在代码编辑器中打开

```bash
brev open sample-reason2-bdb1b0 cursor
```

## 第 3 步：验证 Hugging Face CLI

下载 Cosmos Reason2 模型需要 Hugging Face Token：

```bash
# Authenticate with Hugging Face
~/.local/bin/hf auth login
```

出现提示后，输入你的 Hugging Face token。你可以在 [https://huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) 创建 token。

**重要**：请确保你已获得 [Cosmos-Reason2-8B](https://huggingface.co/nvidia/Cosmos-Reason2-8B) 模型的访问权限。如有需要，请先申请访问。

## 第 4 步：运行推理和后训练

现在你已经可以开始使用 Cosmos Reason2 进行推理了！

按照 [Cosmos Reason GitHub 仓库](https://github.com/nvidia-cosmos/cosmos-reason2) 中提供的步骤运行推理和后训练示例。

## 故障排查

### 模型下载问题

如果模型下载失败：

1. 验证你的 Hugging Face 登录状态：`~/.local/bin/hf whoami`
2. 确认你拥有 Cosmos-Reason2-8B 模型的访问权限
3. 检查你的网络连接
4. 尝试手动下载：`huggingface-cli download nvidia/Cosmos-Reason2-8B`

### SSH 连接问题

如果 SSH 连接断开：

1. Brev 实例可能会在空闲后暂停
2. 在 Brev 控制台中查看实例状态
3. 如有需要，重启实例
4. 使用 SSH 命令重新连接

## 资源管理

### 停止实例

为避免产生不必要的费用：

1. 打开你的 Brev 控制台
2. 选择你的实例
3. 使用完成后点击 **"Stop"** 或 **"Delete"**

### 保存你的工作

在停止实例之前：

```bash
# Save model checkpoints to cloud storage (e.g., S3, GCS)
# Or download them to your local machine
scp -r ubuntu@<your-instance-ip>:~/cosmos-reason2/examples/post_training_hf/outputs ./local-outputs
```

## 其他资源

- [Cosmos Reason 2 GitHub 仓库](https://github.com/nvidia-cosmos/cosmos-reason2)
- [Hugging Face 上的 Cosmos Reason 2 模型](https://huggingface.co/nvidia/Cosmos-Reason2-8B)
- [Brev 文档](https://docs.brev.dev)
- [Cosmos Cookbook](https://github.com/nvidia-cosmos/cosmos-cookbook)

## 支持

如遇以下相关问题：

- **Cosmos Reason 2**：在 [GitHub 仓库](https://github.com/nvidia-cosmos/cosmos-reason2/issues) 中提交 issue
- **Brev Platform**：联系 [Brev support](https://brev.dev/support)
