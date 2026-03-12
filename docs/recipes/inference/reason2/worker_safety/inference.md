# 使用 Cosmos Reason 2 在传统仓库中保障工人安全

> **作者：** [Paula Ramos, PhD](https://www.linkedin.com/in/paula-ramos-phd/)
> **组织：** [NVIDIA](https://nvidia.com/)

## 概览

本配方展示了一个完整的视频推理流水线，使用 [**Cosmos Reason 2**](https://github.com/nvidia-cosmos/cosmos-reason2) 和 [FiftyOne](https://github.com/voxel51/fiftyone)，在具有挑战性的 **“brownfield”** 环境中实现工业安全巡检自动化。

它展示了如何提示视频语言模型（VLM）忽略环境噪声（如褪色油漆或老旧机械），并严格依据 Video Dataset for Safe and Unsafe Behaviours 中定义的视觉真值来分类工人行为。

主笔记本：[Worker Safety notebook](worker_safety.ipynb)
环境设置指南：[环境设置与系统要求](setup.md)

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| [**Cosmos Reason 2**](https://github.com/nvidia-cosmos/cosmos-reason2) | 推理 | 零样本安全合规与危险检测 |

<video src="assets/overload_forklift.webm" controls width="720"></video>

*叉车超载示例：当画面中可见 3 个或更多方块时，模型应将其分类为 “Carrying Overload with Forklift”。*

---

## 环境设置

在运行本配方之前，请确保你已完成所需的环境配置。

前置条件：

- 具备 CUDA 支持的 NVIDIA GPU（本配方已在配备 CUDA 13.0 的 NVIDIA RTX PRO 5000 Blackwell GPU 上测试）。
- 已安装 `fiftyone`、`transformers`、`torch` 和 `qwen-vl-utils`。
- 可从 Hugging Face 获取 [`pjramg/Safe_Unsafe_Test`](https://huggingface.co/datasets/pjramg/Safe_Unsafe_Test) 数据集。

---

## 动机：“传统”仓库中的安全问题

现代工厂通常非常整洁，但许多现实世界设施其实是“传统”仓库——建筑较老、布局不规则、照明较差、基础设施磨损严重。在这些环境中，标准计算机视觉模型往往会失效，因为它们会把褪色的地面标记误判为仍在使用的安全区域。
Safe/Unsafe Behaviours Dataset 正好真实地记录了这种情况。它来自一家金属制造工厂，包含 8 个特定行为类别（4 个安全、4 个不安全），在 39 天内完成采集。我们在本配方中使用的数据集只是原始数据集的一个样本（[论文](https://www.sciencedirect.com/science/article/pii/S235234092400756X)，[数据集](https://data.mendeley.com/datasets/xjmtb22pff/1)）。

该数据集中定义的挑战包括：

- 环境噪声：褪色的黄色线条、未涂装区域以及复杂背景。
- 2D 投影限制：摄像机角度会让工人看起来像是走在通道“上”，而实际上他们可能是在通道旁边行走。
- 严格合规：安全规则是二元的（例如，背心要么穿了，要么没穿），但视觉数据却很杂乱。

解决方案在于：Cosmos Reason 2 让我们无需训练自定义分类器。相反，我们通过 **prompt engineering** 让模型扮演**专家检查员**。我们指示模型忽略 **“old warehouse”** 的外观特征，而严格关注论文中给出的视觉定义——具体来说是 Green Paths、Green Vests 和 Block Counts。

本配方展示了一个可复现的流水线，它可以：

1. 将安全数据集加载到 FiftyOne 中。
2. 基于数据集真值构建上下文感知提示。
3. 对视频片段运行 Cosmos Reason 2 推理。
4. 解析结构化 JSON 输出以进行危险检测。
5. 可视化结果并与真值进行比较。

---

## 流水线概览

端到端流程如下：

1. 加载数据：从 Hugging Face 导入 [`Safe_Unsafe_Test`](https://huggingface.co/datasets/pjramg/Safe_Unsafe_Test) 数据集。
2. 定义提示：将数据集中具体的视觉规则编码为 system prompt 和 user prompt。
3. 运行推理：通过 Hugging Face Transformers 使用 [`Cosmos-Reason2-2B`](https://huggingface.co/nvidia/Cosmos-Reason2-2B) 处理视频。
4. 解析输出：提取 JSON 预测结果（Class ID、Label、Rationale）。
5. 可视化：在 FiftyOne 中探索结果，验证在“old warehouse”约束下的准确性。

---

## 1. 加载数据集

我们首先将 Hugging Face Hub 上的数据集加载到 FiftyOne 中。该数据集包含 Full HD（1920x1080）视频片段，帧率为 24 fps，长度从 1 秒到 20 秒不等。

```python
import fiftyone as fo
import fiftyone.utils.huggingface as fouh

# Load the dataset (persistent=True ensures we don't re-download on every run)
dataset = fo.load_dataset("pjramg/Safe_Unsafe_Test")

# Verify the first sample is a video
sample = dataset.first()
print(f"Loaded dataset with {len(dataset)} samples. Media type: {sample.media_type}")
```

---

## 2. “专家检查员”提示策略

这是最关键的一步。由于设施较旧，我们不能笼统地要求模型寻找“安全区域”。我们必须将系统指令映射到论文中定义的数据集特定约束上。

System prompt（角色与约束）：我们指示模型扮演工业安全检查员。关键在于，我们加入了负向约束，以应对数据集的限制：

- 忽略背景：不要仅因褪色油漆或设施老旧失修就判定为危险。
- 忽略坐着的工人：“intervention” 类别仅适用于站在机器面板前的工人；坐着的工人（或驾驶员）属于背景噪声。
- 优先级：如果同时存在安全和不安全行为，优先判定不安全行为。

User prompt（严格分类表）：我们向模型提供数据集论文表 1 中的精确定义：

| 编号 | 标签 | 视觉定义（Ground Truth） |
| :--- | :--- | :--- |
| 0 | Safe Walkway Violation | Walking OUTSIDE the Green Path. |
| 4 | Safe Walkway | Walking INSIDE the Green Path. |
| 1 | Unauthorized Intervention | Interacting with machine board WITHOUT a green vest. |
| 3 | Carrying Overload with Forklift | Carrying 3 or more blocks. |

```python
SYSTEM_INSTRUCTIONS = """
You are an expert Industrial Safety Inspector monitoring a manufacturing facility.
Your goal is to classify the video into EXACTLY ONE of the 8 classes defined below.

CRITICAL NEGATIVE CONSTRAINTS (What to IGNORE):
1. IGNORE SITTING WORKERS:
   - If a person is SITTING at a machine board working, this is NOT an intervention class. Ignore them.
   - If a person is SITTING driving a forklift, the driver is NOT the class. Focus only on the LOAD carried.
2. IGNORE BACKGROUND:
   - The facility is old. Do not report hazards based on faded floor markings or unpainted areas.
3. SINGLE OUTPUT:
   - Even if multiple things happen, choose the MOST PROMINENT behavior.
   - Prioritize UNSAFE behaviors over SAFE behaviors if both are present.
"""

USER_PROMPT_CONTENT = """
Analyze the video and output a JSON object. You MUST select the class ID and Label EXACTLY from the table below.

STRICT CLASSIFICATION TABLE (Use these exact IDs and Labels):

| 编号 | 标签 | 定义（Ground Truth） | 危险状态 |
| :--- | :--- | :--- | :--- |
| 0 | Safe Walkway Violation | Worker walks OUTSIDE the designated Green Path. | TRUE (Unsafe) |
| 4 | Safe Walkway | Worker walks INSIDE the designated Green Path. | FALSE (Safe) |
| 1 | Unauthorized Intervention | Worker interacts with machine board WITHOUT a green vest. | TRUE (Unsafe) |
| 5 | Authorized Intervention | Worker interacts with machine board WITH a green vest. | FALSE (Safe) |
| 2 | Opened Panel Cover | Machine panel cover is left OPEN after intervention. | TRUE (Unsafe) |
| 6 | Closed Panel Cover | Machine panel cover is CLOSED after intervention. | FALSE (Safe) |
| 3 | Carrying Overload with Forklift | Forklift carries 3 OR MORE blocks. | TRUE (Unsafe) |
| 7 | Safe Carrying | Forklift carries 2 OR FEWER blocks. | FALSE (Safe) |

INSTRUCTIONS:
1. Identify the behavior in the video.
2. Match it to one row in the table above.
3. Output the exact "ID" and "Label" from that row. Do not invent new labels like "safe and compliant".

OUTPUT FORMAT:
{
  "prediction_class_id": [Integer from Table],
  "prediction_label": "[Exact String from Table]",
  "video_description": "[Concise description of the observed action]",
  "hazard_detection": {
    "is_hazardous": [true/false based on the Hazard Status column],
    "temporal_segment": "[Start Time - End Time] or null"
  }
}
"""
```

---

## 3. 运行 Cosmos Reason 2 推理

我们使用 transformers 库运行模型。注意，我们定义了一种对话结构，其中视频输入位于文本提示之前，这与模型训练时的输入形式一致。

```python
import torch
import transformers

def run_inference(model, processor, video_path):
    conversation = [
        {"role": "system", "content": [{"type": "text", "text": SYSTEM_INSTRUCTIONS}]},
        {
            "role": "user",
            "content": [
                {"type": "video", "video": video_path},
                {"type": "text", "text": USER_PROMPT_CONTENT},
            ],
        },
    ]

    inputs = processor.apply_chat_template(
        conversation,
        tokenize=True,
        add_generation_prompt=True,
        return_tensors="pt",
        fps=4,
    ).to(model.device)

    generated_ids = model.generate(**inputs, max_new_tokens=1024)
    # ... decode output ...
```

---

## 4. 解析并存储结果

模型会返回一个结构化 JSON 对象，其中包含 `prediction_class_id`、`prediction_label` 和 `hazard_detection` 标志。我们解析该字符串，并将其直接存储到 FiftyOne sample 中。
这样一来，后续就可以利用 FiftyOne 强大的筛选能力——例如，单独筛出所有 “Class 3”（Forklift Overload）预测，以验证模型是否正确统计了方块数量。

```python
try:
    # Clean and parse the JSON output
    clean_json = output_text.strip().replace("```json", "").replace("```", "")
    json_data = json.loads(clean_json)

    # Store in FiftyOne
    sample["cosmos_analysis"] = json_data
    sample["safety_label"] = fo.Classification(label=json_data.get("prediction_label"))
    sample.save()

except Exception as e:
    print(f"JSON Parsing failed: {e}")
```

---

## 5. 结果与可视化

启动 FiftyOne App 来审查该“检查员”的表现。

```python
session = fo.launch_app(dataset)
session.wait()
```

可重点关注以下方面：

- 绿色通道合规性：检查模型是否能基于绿色标记正确区分 Class 0 和 Class 4，即使地面油漆已经褪色。
- 背心检测：验证 Class 1（Unauthorized）是否只会在面板前站立且未穿绿色背心的工人出现时触发。
- 叉车计数：确保 Class 3（Overload）仅严格应用于负载 3 个以上方块的情况，而 2 个方块仍归为 Class 7（Safe）。

观察：由于提示中加入了负向约束，你可能会注意到模型能够正确忽略坐着驾驶叉车的工人，只关注负载本身。

示例结果片段：

<video src="assets/unauthorized_intervention.webm" controls width="720"></video>

*未授权干预：站在机器面板前的工人未穿绿色背心。*

<video src="assets/safe_walkway_violation.webm" controls width="720"></video>

*安全通道违规：工人走在指定绿色通道之外。*

<video src="assets/overload_forklift.webm" controls width="720"></video>

*叉车超载：叉车承载了 3 个或更多方块。*

---

## 结论

本配方展示了：

- 如何在不进行微调的情况下，将 Cosmos Reason 2 适配到专业工业领域。
- prompt engineering 在克服 “brownfield” 环境噪声中的重要性。
- 如何把学术数据集中的定义（例如 Safe and Unsafe Behaviours 论文中的定义）映射为可执行的模型约束。

这种方法可以推广到：

- 建筑工地监测（PPE 检测）。
- 零售防损。
- 物流与库存审计。

如需获取完整代码并亲自运行此分析，请确认你的环境中可以访问 `pjramg/Safe_Unsafe_Test` 数据集和 Cosmos-Reason2 模型。

在主笔记本中运行完整工作流：[Worker Safety notebook](worker_safety.ipynb)。

### 参考资料

- Safe and Unsafe Behaviours Dataset：[Mendeley Data](https://data.mendeley.com/datasets/xjmtb22pff/1)
- 原始论文：Fernandes, P., et al. (2024). "Video Dataset for Safe and Unsafe Behaviours Detection in Industrial Environments." *Data in Brief*. [DOI: 10.1016/j.dib.2024.111258](https://www.sciencedirect.com/science/article/pii/S235234092400756X)

---

## 文档信息

**发布日期：** 2026 年 2 月 4 日

### 引用

如果你使用了本配方或参考了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_worker_safety_2026,
  title={Worker Safety in a Classical Warehouse with Cosmos Reason 2},
  author={Ramos, Paula},
  organization={NVIDIA},
  year={2026},
  month={February},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/reason2/worker_safety/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```
