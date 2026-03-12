# 环境设置与系统要求

本指南总结了在配备 **NVIDIA GPU** 的 **Ubuntu Linux** 上安装 **Cosmos Reason2**、使用 **uv** 进行依赖管理，以及通过带有专用内核的 **JupyterLab** 运行笔记本的设置流程。

---

## 1) 系统前置条件（Ubuntu）

安装所需的系统包：

```bash
sudo apt-get update
sudo apt-get install -y curl ffmpeg git git-lfs
git lfs install
```

验证你的 GPU、NVIDIA 驱动和 CUDA 支持：

```bash
nvidia-smi
```

> **注意**：本配方在一台运行 **Ubuntu 24.04**、支持 **CUDA 13.0** 的 **NVIDIA RTX PRO 5000 Blackwell GPU** 笔记本上进行了测试。配置时我们使用 **cu130** 环境 extra。你可能需要根据自己的系统调整 CUDA 版本。

---

## 2) 安装 `uv`（每位用户一次）

安装 **uv**（Astral）：

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
uv --version
```

---

## 3) 克隆 Cosmos Reason2 仓库

```bash
REPO_ROOT=/path/to/your/preferred/projects/directory  # e.g., $HOME/Documents/GitHub or $HOME/projects
mkdir -p "$REPO_ROOT"
cd "$REPO_ROOT"
git clone https://github.com/nvidia-cosmos/cosmos-reason2.git
cd cosmos-reason2
git lfs pull
```

---

## 4) 使用 Hugging Face 进行认证（模型下载所需）

```bash
uvx hf auth login
```

---

## 5) 创建 Cosmos Reason2 环境（CUDA 13.0）

创建环境并安装依赖（这会在仓库内创建 `./.venv`）：

```bash
uv sync --extra cu130
```

激活环境：

```bash
source .venv/bin/activate
```

验证 PyTorch 和 CUDA 是否可用：

```bash
python -c "import torch; print(torch.__version__); print('cuda', torch.cuda.is_available())"
```

预期结果：

- `cuda True`

---

## 6) 在同一环境中安装额外软件包

### 安装 FiftyOne

```bash
pip install -U fiftyone
```

### 安装 JupyterLab 与内核支持

```bash
pip install -U jupyterlab ipykernel
```

---

## 7) 将该环境注册为 Jupyter 内核（可在 JupyterLab 中找到）

注册该环境，使其显示为可选内核：

```bash
python -m ipykernel install --user --name cosmos-reason2 --display-name "Python (cosmos-reason2)"
```

确认它已存在：

```bash
jupyter kernelspec list
```

你应当能看到类似如下内容：

```
cosmos-reason2    /home/<user>/.local/share/jupyter/kernels/cosmos-reason2
```

---

## 8) 运行 JupyterLab 并选择正确的内核

启动 JupyterLab（建议在仓库根目录运行）：

```bash
cd ~/Documents/GitHub/cosmos-reason2
source .venv/bin/activate
jupyter lab
```

在 JupyterLab 界面中：

- **Kernel → Change Kernel → `Python (cosmos-reason2)`**

---

## 9) 在笔记本内部运行脚本（重要）

如果你的笔记本位于：

```
cosmos-reason2/notebooks/
```

并且你运行：

```python
!python ../scripts/inference_sample.py
```

脚本将相对于笔记本目录运行，这会影响路径解析。

### 推荐方式（从仓库根目录运行）

在一行中从仓库根目录运行脚本：

```python
!cd .. && python scripts/inference_sample.py
```

或者分两步：

```python
%cd ..
!python scripts/inference_sample.py
```

这样可以确保脚本使用正确的仓库相对路径。

---

## 快速故障排查说明

- 如果发生端口冲突（例如 `Address already in use`），请尝试为本地服务使用其他端口。
- 如果视频解码失败，请确保系统已安装 `ffmpeg`，并可通过 `ffmpeg -version` 调用。

---

**完成！** 现在你已经拥有：

- 一个专用的 `cosmos-reason2` Python 环境（`.venv`）
- Cosmos Reason2 以及从 Hugging Face 获得授权的所有模型（依据 Cosmos Reason2 仓库）
- 安装在该环境中的 FiftyOne
- 安装在该环境中的 JupyterLab
- 可在 JupyterLab 中选择的内核：**Python (cosmos-reason2)**
