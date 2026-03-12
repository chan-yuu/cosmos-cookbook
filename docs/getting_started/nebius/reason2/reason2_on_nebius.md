# 在 Nebius 上开始使用 Cosmos Reason 2：推理
>
> **作者：** [Jathavan Sriram](https://www.linkedin.com/in/jathavansriram)
> **组织：** NVIDIA

[Cosmos Reason 2](https://huggingface.co/nvidia/Cosmos-Reason2-8B) 是 NVIDIA 开源的视觉-语言推理模型，专为机器人、自动驾驶和物理世界理解而设计。本指南将向你展示如何使用 [Containers over VM](https://docs.nebius.com/compute/virtual-machines/containers) 功能，将其部署到 [Nebius AI Cloud](https://nebius.com/) 上。该功能可提供基于按需云 GPU 的 API 端点，并且只需极少设置。

## 概览

完成本指南后，你将拥有一个可用的 Cosmos Reason 2 API 端点，能够：

- 使用自然语言查询分析图像和视频
- 生成时间戳和结构化 JSON 输出
- 预测机器人动作和 2D 轨迹
- 评估合成数据是否符合物理规律

| | |
|---|---|
| **完成时间** | ~15 分钟 |
| **模型选项** | 2B（更快，约 5GB VRAM）或 8B（能力更强，约 17GB VRAM） |
| **GPU 要求** | L40S、H100 或同等级别 |

## 前置条件

- 一个 Nebius 账户（[在此注册](https://nebius.com/)）
- 一个 [Hugging Face](https://huggingface.co/) 账户，并已获得 [Cosmos Reason 2](https://huggingface.co/nvidia/Cosmos-Reason2-8B) 的访问权限
- Python 3.10+（用于运行推理示例）
- `jq` 命令行工具（用于在 curl 示例中解析 JSON 响应）- [安装指南](https://jqlang.github.io/jq/download/)

## 第 1 步：创建一个运行 Cosmos Reason 2 的 Nebius Container over VM 实例

[Containers over VMs](https://docs.nebius.com/compute/virtual-machines/containers) 让你可以部署一个预装 vLLM 的 GPU 虚拟机——无需手动设置 Docker。该 VM 会自动获得一个公有 IP，以便立即进行 API 访问。

1. 登录你的 [Nebius AI Cloud 账户](https://console.nebius.com/)

2. 导航到 **Containers over VM** 部分

    ![Containers over VM](./images/nebius-02-container-over-vm.png)

3. 点击 **Create container over VM** 创建新的虚拟机

    ![Create container over VM](./images/nebius-03-container-over-vm-button.png)

4. 配置 **Project, Name and Container Image**
    - 在目标区域中选择 **Project**。请注意，不同区域可用的 GPU 类型可能不同。你可以在 [Nebius documentation](https://docs.nebius.com/compute/virtual-machines/types) 中了解更多信息。
    - 为你的 VM 提供一个 **Name**
    - 选择 **vLLM+Jupyter** 镜像预设

    ![Configure Main Settings](./images/nebius-04-container-over-vm-config.png)

5. 配置 **vLLM Model Name**
    - 将 vLLM Model Name 设置为以下之一：
      - `nvidia/Cosmos-Reason2-2B`
      - `nvidia/Cosmos-Reason2-8B`

    ![Configure vLLM Model](./images/nebius-05-container-over-vm-config-vllm-model.png)

6. 配置 **Hugging Face Token**

    在此输入你的 Hugging Face token。你可以在 [Hugging Face Account Settings](https://huggingface.co/settings/tokens) 中创建 token。如需进一步了解，请参阅 [Hugging Face Documentation](https://huggingface.co/docs/hub/en/security-tokens)。

    **重要**：请确保你的 token 具备对 [Cosmos-Reason2-8B](https://huggingface.co/nvidia/Cosmos-Reason2-8B) 和 [Cosmos-Reason2-2B](https://huggingface.co/nvidia/Cosmos-Reason2-2B) 仓库的读取权限。

    ![HF Token](./images/nebius-06-container-over-vm-config-hf-token.png)

7. 选择 **Computing resources**

    在这一步中，你需要选择希望运行的 GPU 实例。若要了解支持哪些 GPU，请访问 [Supported Models Documentation](https://docs.nvidia.com/nim/vision-language-models/latest/support-matrix.html#cosmos-reason2)。

    下面的示例中选择了一个 L40S 实例。

    ![GPU Instance Selection](./images/nebius-07-container-over-vm-config-gpu-selection.png)

8. **Additional Settings**

    保持默认预设即可。如有需要，你也可以进一步配置 SSH 密钥和网络等设置，或者直接使用默认值。

    ![Additional Configuration](./images/nebius-08-container-over-vm-config-advanced.png)

9. **确认并创建 VM**

    在配置页面底部点击 **Create container over VM** 按钮来创建实例。

    ![Create](./images/nebius-09-container-over-vm-config-create-button.png)

10. **等待环境就绪**

    Cosmos 的 VM 和容器大约需要 5-10 分钟才能变为可用。状态会从 `Pulling` 变为 `Container Running.`

    ![Pulling Status](./images/nebius-10-container-over-vm-status-running.png)

## 第 2 步：使用 Cosmos Reason 2 运行推理

当你的实例显示为 `Container Running` 后，就可以开始发起 API 调用了。首先，你需要从 Nebius 控制台获取端点 URL 和 API key，然后通过一个简单的 curl 命令测试连接。

> **注意：** 容器启动后，vLLM 服务器可能还需要额外 1-2 分钟完成初始化。如果你遇到连接错误，请稍等后重试。

1. **获取 vLLM Endpoint**
    - 在 Nebius Web Console 中进入已创建的 VM
    - 在顶部点击 **Endpoints**
    - 选择 `Copy vLLM API endpoint URL` 以获取 Endpoint URL
    - Endpoint URL 的格式为 `PUBLICIP:PORT`

    ![vLLM Endpoint](./images/nebius-11-vllm-endpoint.png)

2. **获取 vLLM API Key**
    - 查找 **Container Parameters** 部分
    - 复制 **vLLM API Key**

    ![vLLM API Key](./images/nebius-12-vllm-token.png)

### 基本连接测试

将你的凭据导出为环境变量，然后使用一个简单查询测试端点：

```bash
# Replace with your actual endpoint values from the Nebius Console:
export VLLM_ENDPOINT="PUBLICIP:8000"     # Format: PUBLIC_IP:PORT
export VLLM_API_KEY="YOUR_API_KEY"         # Copy from vLLM API Key

# You can now reference these variables in your curl command:
curl -s http://$VLLM_ENDPOINT/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $VLLM_API_KEY" \
  -d '{
    "model": "nvidia/Cosmos-Reason2-2B",
    "messages": [
      {"role": "user", "content": "What is a robot?"}
    ],
    "max_tokens": 512
  }' | jq -r '.choices[0].message.content'
```

你应当看到类似如下的输出：

```text
A robot is an automated machine capable of performing tasks that typically require human intelligence, such as recognizing objects, understanding language, or adapting to changing environments. Robots can be programmed to mimic human actions, learn from experience, and even solve problems autonomously. They are widely used in industries like manufacturing, healthcare, and service sectors to enhance efficiency and safety.
```

### 运行 Cosmos Reason 2 推理示例

本节提供了一个 Python 脚本，用于运行多个 Cosmos Reason 2 提示测试，展示图像/视频理解、时间定位、机器人推理以及合成数据评估等多种能力。

#### 设置本地环境

首先，在本地机器上克隆仓库并安装依赖：

```bash
# Clone the repository (if you haven't already)
git clone https://github.com/nvidia-cosmos/cosmos-cookbook.git
cd cosmos-cookbook/docs/getting_started/nebius/reason2/src

# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

#### 配置环境变量

使用 Nebius Console 中的值设置连接信息：

```bash
# Set your vLLM endpoint and API key
export VLLM_ENDPOINT="YOUR_PUBLIC_IP:8000"  # e.g., "89.169.115.247:8000"
export VLLM_API_KEY="YOUR_VLLM_API_KEY"
```

#### 运行测试脚本

`cosmos_reason2_tests.py` 脚本包含多个测试用例，涵盖 Cosmos Reason 2 的不同能力：

| Test ID | Test Name | Description |
|---------|-----------|-------------|
| `1_basic_image` | Basic Image Understanding | Simple image description |
| `2_basic_video` | Basic Video Understanding | Simple video captioning |
| `3_temporal_localization` | Temporal Localization | Video with timestamps (mm:ss) |
| `4_temporal_json` | Temporal JSON Output | Video events in JSON format |
| `5_robotics_next_action` | Robotics Next Action | Predict robot's next action |
| `6_2d_trajectory` | 2D Trajectory Creation | Generate gripper trajectory coordinates |
| `7_sdg_critic` | SDG Critic | Evaluate video for physics adherence |
| `8_2d_grounding` | 2D Object Grounding | Locate objects with bounding boxes |

**运行所有测试：**

```bash
python cosmos_reason2_tests.py
```

**运行特定测试：**

```bash
# Run only image and video basic tests
python cosmos_reason2_tests.py --tests 1_basic_image 2_basic_video

# Run robotics-related tests
python cosmos_reason2_tests.py --tests 5_robotics_next_action 6_2d_trajectory
```

**列出可用测试：**

```bash
python cosmos_reason2_tests.py --list
```

**使用命令行参数而不是环境变量：**

```bash
python cosmos_reason2_tests.py \
    --host 89.169.115.247 \
    --port 8000 \
    --api-key YOUR_API_KEY \
    --tests all
```

#### 查看结果

脚本会输出每个测试的结果，包括：

- 发送给模型的提示词
- 模型的响应
- Token 使用统计

示例输出：

```text
======================================================================
Test: Basic Image Understanding
Description: Simple image description without reasoning
======================================================================

Media URL: https://assets.ngc.nvidia.com/products/api-catalog/cosmos-reason1-7b/critic_rejection_sampling.jpg
Prompt: What is in this image? Describe the objects and their positions.

Waiting for response...

----------------------------------------------------------------------
Response:
----------------------------------------------------------------------
The image shows a tabletop scene with several objects arranged on it...

[Tokens - Prompt: 1234, Completion: 256, Total: 1490]
```

#### 关键提示模式

测试脚本演示了来自 [Cosmos Reason 2 Prompt Guide](../../prompt_guide/reason_guide.md) 的重要提示模式：

1. **Media-First Ordering**：在消息内容中，图像/视频应出现在文本之前
2. **Reasoning Mode**：使用 `<think>...</think>` 标签启用 chain-of-thought 推理
3. **Structured Output**：请求 JSON 格式等机器可读输出
4. **Sampling Parameters**：根据任务要求调整 temperature 和 top_p

有关如何为 Cosmos Reason 2 编写提示的更多细节，请参阅完整的 [Prompt Guide](../../prompt_guide/reason_guide.md)。

## 故障排查

### 模型下载问题

如果模型下载失败：

1. 验证你的 Hugging Face 登录状态：`huggingface-cli whoami`
2. 确认你已在 [Cosmos-Reason2-8B](https://huggingface.co/nvidia/Cosmos-Reason2-8B) 上接受模型许可协议
3. 检查你的网络连接
4. 尝试手动下载：`huggingface-cli download nvidia/Cosmos-Reason2-8B`

### Connection Refused

如果你收到 “connection refused” 错误：

1. 确认 VM 状态在 Nebius 控制台中显示为 `Container Running`
2. 启动后等待 2-3 分钟，让 vLLM 完成初始化
3. 确认端点 URL 格式为：`http://PUBLIC_IP:8000`（不是 `https`）
4. 检查 8000 端口是否可访问（Nebius 默认会打开该端口）

### 401 Unauthorized

如果你收到认证错误：

1. 确认你使用的是 Container Parameters 中正确的 **vLLM API Key**（不是你的 HuggingFace token）
2. 验证请求中的 `Authorization: Bearer YOUR_KEY` header 格式是否正确
3. 如有需要，请在 Nebius 控制台中重新生成 API key

### 超时或响应缓慢

如果请求超时或响应很慢：

1. 由于模型预热，第一次请求可能需要 30-60 秒
2. 大图像或长视频会增加处理时间
3. 检查 GPU 显存使用情况——8B 模型比 2B 模型需要更多 VRAM
4. 在开发阶段可考虑使用 2B 模型以获得更快响应

## 资源管理

> **费用警告：** GPU 实例在运行期间会产生费用。不使用时请务必停止或删除实例，以避免产生意外成本。

### 停止实例

1. 前往 [Nebius Console](https://console.nebius.com/) → **Compute** → **Containers over VMs**
2. 在列表中找到你的实例
3. 点击右侧的 **⋮**（三个点）菜单
4. 选择：
   - **Stop** - 暂停实例（保留配置，并停止计算资源计费）
   - **Delete** - 永久删除实例及其数据

### 监控成本

前往 Nebius 控制台中的 **Billing** 页面，监控你的使用情况和支出。

## 其他资源

- [Cosmos Reason 2 GitHub 仓库](https://github.com/nvidia-cosmos/cosmos-reason2)
- [Hugging Face 上的 Cosmos Reason 2 模型](https://huggingface.co/nvidia/Cosmos-Reason2-8B)
- [Nebius Documentation](https://nebius.com/docs)

## 支持

如遇以下相关问题：

- **Cosmos Reason 2**：在 [GitHub 仓库](https://github.com/nvidia-cosmos/cosmos-reason2/issues) 中提交 issue
- **Nebius Platform**：联系 [Nebius support](https://nebius.com/support)
