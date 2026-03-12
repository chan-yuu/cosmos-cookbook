# 使用 Cosmos Reason 的视频搜索与摘要

> **作者：** [Sammy Ochoa](https://www.linkedin.com/in/sammy-ochoa/)
> **组织：** NVIDIA

## 概览

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| [Cosmos Reason 2 8B](https://huggingface.co/nvidia/Cosmos-Reason2-8B)| 推理 | 大规模视频搜索与摘要。 |

海量视频数据包含了理解并优化仓库、工厂、零售门店、城市等场景运营所需的关键信息。无论是归档视频文件还是实时摄像头流，都需要耗时的人工审查，才能从视频中提取有价值的洞察。

NVIDIA 的 [Video Search and Summarization Blueprint](https://build.nvidia.com/nvidia/video-search-and-summarization)（VSS）是一种参考架构，用于结合视觉语言模型、计算机视觉模型和大语言模型来分析和理解海量视频数据。VSS 可以通过多种提示轻松配置，从而根据目标用例调整模型响应。

![VSS UI Example](assets/warehouse_summary_example.png)

VSS 允许用户上传视频文件或连接实时摄像头流，以生成摘要、回答问题，或在关注事件发生时发送告警。

Cosmos Reason 在 VSS 中被用作默认视觉语言模型，以便为输入视频生成高质量描述。VSS 首先将输入视频切分为较小片段（10s-30s），然后把这些片段作为 GPU 优化推理流水线的一部分提供给 Cosmos Reason，从而快速并行地为视频片段生成描述。

除了来自 Cosmos Reason 的描述外，还会将检测数据和音频转录等额外数据源与视频描述结合起来。这些数据会存储到向量数据库和图数据库中，随后由 LLM 检索，以根据用户提示生成摘要、回答问题并触发告警。

NVIDIA Brev Launchable 是快速启动 VSS 预配置环境的一种便捷方式。

- [VSS Brev Launchable](https://docs.nvidia.com/vss/latest/content/cloud_brev.html)

对于 Nebius 部署，建议使用本地部署配置的 8xH100 GPU 实例。或者，也可以按照单 GPU 部署配置，使用单个 1xH100 实例配合更小的模型。

- [多 GPU 本地部署配置](https://docs.nvidia.com/vss/latest/content/vss_dep_docker_compose_x86.html#local-deployment)
- [单 GPU 本地部署配置](https://docs.nvidia.com/vss/latest/content/vss_dep_docker_compose_x86.html#fully-local-deployment-single-gpu)

对于本地系统或其他云实例上的自定义部署，VSS 文档提供了适用于多种硬件配置的若干部署方案。

- [环境设置与系统要求](https://docs.nvidia.com/vss/latest/content/prereqs_x86.html#)

## 关键特性

- **视频摘要**：根据用户提示生成自定义视频摘要。
- **视频问答**：基于视频文件和流，使用高级 Graph-RAG 技术回答问题。
- **直播告警**：当实时视频中出现感兴趣事件时接收告警。

## VSS Blueprint 架构

![VSS Architecture](assets/vss_architecture.jpg)

VSS 由两个主要部分组成：

- **摄取流水线**：从输入视频中提取视觉洞察，
  形式包括描述和场景说明。
- **检索流水线**：对摄取流水线中的视觉洞察进一步处理、索引，
  并用于摘要、问答和告警等检索任务。

### 摄取流水线

摄取流水线既支持离线和批量处理视频/图像文件，
也支持对来自摄像头的实时流进行在线处理。

- 视频文件会被划分为较小片段——通常为 10 到 30 秒，具体取决于模型和应用。单个片段的处理会并行分布到多个 GPU 上，以获得更好的性能。

- 对于每个视频片段，会采样固定数量的帧。例如，从一个 10 秒的视频片段中采样 10 帧并提供给 Cosmos Reason 以生成描述。这些数值可根据用例和模型上下文长度进行配置。

- 可选启用基于 Riva ASR 的音频转录，为每个视频片段生成音频转录文本。

- 可启用基于 Grounding Dino 的检测与跟踪流水线，以获得关于视频中特定对象的更多洞察。检测和跟踪数据会叠加到视频上，然后提供给 Cosmos Reason，以生成更详细的描述。

- 每个片段的 VLM 描述、音频转录、CV 元数据和时间戳信息都会发送到检索流水线，以进行进一步处理和索引。

### 检索流水线

检索流水线由 [CA-RAG library](https://github.com/NVIDIA/context-aware-rag) 实现，负责处理摄取流水线的输出，并将其用于长视频摘要、实时流摘要以及基于索引数据的问答等多种检索任务。

- VLM 描述、音频转录及其相关元数据会被处理、索引并存储到向量数据库和图数据库中。

- 加速版 NeMo Retriever Embedding NIM 用于对 VLM 描述进行高吞吐量文本嵌入。这些文本嵌入会连同相关元数据一起写入向量数据库。

- 类似 Llama 3.1 70B 的 LLM 用于工具调用、解析 VLM 描述，并生成写入图数据库的插入 API 调用，以构建视频的知识图谱。

- 在摘要任务中，VLM 描述和音频转录会一同汇总，再由 LLM 生成最终聚合摘要。

- 在问答任务中，借助 LLM 工具调用，会从知识图谱和向量数据库中提取与你查询相关的信息。检索到的信息会传递给 NeMo reranking 服务，其输出将作为上下文供 LLM 生成问题答案。

## 快速开始

要将 VSS 与 Cosmos Reason 一起使用，你必须先将其部署到云实例或本地 GPU 上。部署完成后，VSS 提供了用于快速视频摘要的参考前端界面，以及便于与你的自定义应用无缝集成的 REST API 后端。

### 云端部署

- [VSS Brev Launchable](https://docs.nvidia.com/vss/latest/content/cloud_brev.html)

### 本地部署

- [环境设置与系统要求](https://docs.nvidia.com/vss/latest/content/prereqs_x86.html#)

部署完成后，你可以参阅 [UI documentation page](https://docs.nvidia.com/vss/latest/content/ui_app.html)，了解如何使用参考 UI 快速测试你自己的视频和实时流。

### 示例代码讲解

当你准备围绕 VSS 构建自定义应用时，可以直接访问后端 REST API，而无需使用 UI。

1. 部署完成后，VSS 默认会在 8100 端口提供后端服务。REST API 可通过该端口访问。

1. 导入 requests 库并设置 REST API 路径。完整的 REST API 文档可在[这里](https://docs.nvidia.com/vss/latest/content/API_doc.html)找到。

    ```
    import requests

    vss_host = "http://localhost:8100"
    files_endpoint = vss_host + "/files" #upload and manage files
    summarize_endpoint = vss_host + "/summarize" #summarize uploaded content
    qna_endpoint = vss_host + "/chat/completions" #ask questions for ingested video
    ```

1. 将视频文件上传到 VSS，并接收一个视频 ID。

    ```
    video_file_path = "/path/to/your/video.mp4"

    with open(video_file_path, "rb") as file:
        files = {"file": ("video_file", file)} #provide the file content along with a file name
        data = {"purpose":"vision", "media_type":"video"}
        response = requests.post(files_endpoint, data=data, files=files) #post file upload request
        response = response.json()

    video_id = response["id"] #save file ID for summarization request
    ```

1. 有了 video id 后，就可以发送摘要请求，并附带用于控制 VLM 描述和输出摘要的提示。

    ```
    body = {
        "id": video_id, #id of file returned after upload
        "prompt": "Write a detailed caption based on the video clip.",
        "caption_summarization_prompt": "Combine sequential captions to create more concise descriptions.",
        "summary_aggregation_prompt": "Write a detailed and well formatted summary of the video captions.",
        "model": "cosmos-reason2",
        "max_tokens": 1024,
        "temperature": 0.3,
        "top_p": 0.3,
        "chunk_duration": 20,
    }

    response = requests.post(summarize_endpoint, json=body)
    response = response.json()
    summary = response["choices"][0]["message"]["content"]
    print(summary)
    ```

1. 视频完成摄取后，还可以发送额外的问答请求，就视频内容提问。

    ```
    question = "What did you see in the video?"

    payload = {
            "id": video_id,
            "messages": [{"content": question, "role": "user"}],
            "model": "cosmos-reason2"
        }

    response = requests.post(qna_endpoint, json=payload)
    response_data = response.json()
    answer = response_data["choices"][0]["message"]["content"]
    print(answer)
    ```

## 结论

Cosmos Reason 在 VSS 中通过优化的 GPU 加速推理流水线生成高质量视频描述。这些描述随后结合嵌入模型和大语言模型进行分析与索引，将关键信息存储到向量数据库和图数据库中，以支持长视频摘要、问答和实时流告警。

VSS 可以通过 [Brev Launchable](https://docs.nvidia.com/vss/latest/content/cloud_brev.html) 轻松部署，也可以遵循本地部署指南完成部署。部署完成后，可以使用参考 Web UI 快速测试自定义视频和提示。若要集成到你自己的应用中，可通过编程方式调用 [VSS REST APIs](https://docs.nvidia.com/vss/latest/content/API_doc.html) 以访问 VSS 的全部功能。

## 资源

- **[VSS Documentation](https://docs.nvidia.com/vss/latest/index.html)** - VSS 主文档
- **[VSS Github Repository](https://github.com/NVIDIA-AI-Blueprints/video-search-and-summarization)** - VSS 开源 GitHub 仓库

---

## 文档信息

**发布日期：** 2026 年 1 月 29 日

### 引用

如果你使用了本配方或参考了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_video_search_and_2026,
  title={Video Search and Summarization with Cosmos Reason},
  author={Ochoa, Sammy},
  organization={NVIDIA},
  year={2026},
  month={January},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/reason2/vss/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的正文引用：**

> Sammy Ochoa (2026). Video Search and Summarization with Cosmos Reason. In *NVIDIA Cosmos Cookbook*. NVIDIA. Accessible at <https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/reason2/vss/inference.html>
