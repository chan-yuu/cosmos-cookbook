# 使用 Cosmos-Reason2-8B 进行第一视角社交与物理推理

> **作者：** [Tingyu Zhang](https://www.linkedin.com/in/tingyu-zhang-b3357314a) • [Sharon Yang](http://linkedin.com/in/sharonxueyangintel)
> **组织：** [IntBot Inc.](https://www.intbot.ai/)

| **模型** | **工作负载** | **用例** |
|-----------|--------------|--------------|
| [Cosmos-Reason2-8B](https://github.com/nvidia-cosmos/cosmos-reason2) | 推理 | 面向机器人的第一视角社交与物理推理 |

- [环境设置与系统要求](https://github.com/nvidia-cosmos/cosmos-reason2#setup)

## 1. 概览

[IntBot](https://www.intbot.ai/) 是一家机器人初创公司，致力于打造具备社交智能的人形机器人，以应对酒店、机场等服务行业的结构性劳动力短缺问题。IntBot 机器人能够以温暖、类人的存在感和富有表现力的肢体语言处理常规问询，从而提升宾客体验；它们可以用 50 多种语言进行自然对话，并提供准确的答案、指引和本地推荐。IntBot 机器人可 24/7 全天候工作，并可根据不同品牌进行定制，能够可靠地承担重复性和持续在线的服务需求，让人工员工把精力集中在更重要的任务上。

<table>
  <tr>
    <td align="center"><img src="./assets/IntBot-GTC.jpg" width="400"><br><em>IntBot 亮相 NVIDIA GTC</em></td>
    <td align="center"><img src="./assets/IntBot-GTC-reception.jpg" width="400"><br><em>IntBot 在 NVIDIA 展台接待区</em></td>
  </tr>
</table>

为了在真实服务场景中发挥作用，IntBot 的机器人必须持续感知、理解并响应各种各样的人类**动作、意图与社交线索**，其中包括：

- **问候与手势** —— 检测并恰当地回应挥手、碰拳、握手及其他邀请信号
- **共享注意力管理** —— 同时与多人保持互动，并判断机器人在任意时刻应该回应谁
- **社交情境边界** —— 理解何时*不应*介入，例如当两位宾客已经在彼此交谈，或打断他们在社交上并不合适时
- **物体交接** —— 识别某人何时正在递出物品（例如文件、证件、宣传资料）*（未来工作）*
- **情绪状态感知** —— 推断宾客何时显得困惑、沮丧、游离或需要帮助 *（未来工作）*

这些能力共同实现了具备社交感知、上下文敏感的交互，超越了脚本化对话；这对于在真实公共环境中大规模部署机器人至关重要，也正是本次评估的核心动机。下述测试重点关注问候/手势、物体运动、共享注意力和社交边界；物体交接和情绪状态感知计划在未来评估中展开。

本配方在一组有针对性的第一视角视频测试上评估 **[Cosmos-Reason2-8B](https://github.com/nvidia-cosmos/cosmos-reason2)**，旨在探查与机器人相关的推理能力，包括：

- 面向机器人的人类意图
- 相对于机器人的物体运动
- 多人之间的空间关系
- 社交情境感知（何时不应参与）

其性能将与 **Qwen3-VL-8B-Instruct** 直接比较；后者是 Cosmos-Reason2-8B 所基于的视觉语言模型。两者使用完全相同的输入和提示。目标是隔离 Cosmos 特有的物理与社交推理对齐所带来的影响，而不是模型规模或原始视觉识别能力差异。

## 2. 实验设置

### 输入

- 短 RGB 视频（2–10 秒）
- 第一视角相机视角（“the camera view is my view”）
- 明确从机器人视角出发的自然语言提示

### 评估模型

- [Cosmos-Reason2-8B](https://github.com/nvidia-cosmos/cosmos-reason2)
- [Qwen3-VL-8B-Instruct](https://github.com/QwenLM/Qwen3-VL)

### 控制变量

两个模型在评估时均使用：

- 相同的视频片段
- 相同的提示
- 相同的推理流水线

这种受控设置确保观察到的差异反映的是推理行为，而不是实验方差。

## 3. 评估结果

> **注意：** 所有测试图像均从第一视角视频录像中提取。为保护隐私，面部已做模糊处理。

### 3.1 面向机器人的人类意图（碰拳）

#### 场景

一个人朝向机器人做出碰拳手势。

<img src="./assets/Test1-fistbump.jpeg" width="400">

示例帧：某人朝向机器人的第一视角发起碰拳

#### 提示

> "Is this person doing a fist bump at me? Can you predict what is the most likely position her fist will be in the next 2 seconds?"

#### 真值

该人确实是在有意与机器人碰拳。

#### 观察到的输出

| 模型 | 结果 |
|-------|---------|
| Cosmos-Reason2-8B | Correctly identifies a fist bump directed at the robot and predicts persistence of the fist-bump pose |
| Qwen3-VL-8B-Instruct | Misclassifies the gesture as pointing and predicts a static, non-interactive posture |

<details>
<summary><strong>模型原始输出</strong></summary>

**Cosmos-Reason2-8B:**
> Yes, the person is doing a fistbump with her right hand. In the next 2 seconds, her fist will likely remain in a fistbump position, maintaining contact with the viewer's fist as she holds the gesture.

**Qwen3-VL-8B-Instruct:**
> No, the person is not doing a fistbump. She is pointing her right index finger forward toward the camera, with her fist clenched but not extended for a fistbump. Her gesture appears to be a direct, assertive, or commanding motion rather than an interactive or celebratory one.
>
> **Most likely position of her fist in the next 2 seconds:**
> Her fist will likely remain clenched and extended forward, continuing to point at the camera. She may slightly adjust her arm's angle or tighten her grip, but the overall position—pointing forward with a clenched fist—will likely persist. There's no indication she will retract her arm or move her fist toward her own body or toward the camera for a fistbump.
>
> In short: **Her fist will stay pointed forward, likely with minor adjustments in angle or tension, but not retracting or moving toward a fistbump.**

</details>

#### 解读

Cosmos-Reason2-8B 能够成功推断出面向机器人的交互式社交意图，而基础模型主要关注手势外观，未能识别互动意图。

### 3.2 物体运动、距离与风险（帽子场景）

七段与帽子相关的视频测试了多种不同的物体运动模式：

- 向远离机器人的侧向投掷
- 朝向机器人的旋转后投掷
- 横向穿过机器人视野的投掷
- 向上抛起后落下
- 投掷前旋转
- 投掷准备姿态
- 投掷后的恢复动作

<table>
  <tr>
    <td align="center"><img src="./assets/Test2-side-throw-away.jpeg" height="180"><br><em>向侧面抛离</em></td>
    <td align="center"><img src="./assets/Test2-Spin-and-Throw.jpeg" height="180"><br><em>旋转并抛掷</em></td>
    <td align="center"><img src="./assets/Test2-Laterial-Throw.jpeg" height="180"><br><em>横向抛掷</em></td>
  </tr>
  <tr>
    <td align="center"><img src="./assets/Test2-Toss-Up.jpeg" height="180"><br><em>向上抛起</em></td>
    <td align="center"><img src="./assets/Test2-Spin.jpeg" height="180"><br><em>旋转中</em></td>
    <td align="center"><img src="./assets/Test2-Throw-Prep.jpeg" height="180"><br><em>抛掷准备</em></td>
  </tr>
  <tr>
    <td align="center" colspan="3"><img src="./assets/Test2-Post-throw-recovery.jpeg" height="180"><br><em>抛掷后恢复</em></td>
  </tr>
</table>

#### 3.2.1 相对机器人轨迹与风险推理

**提示：**

> "Please estimate the landing location of the hat. Is the hat getting closer to me or further away from me? Is there danger for the hat to hit me?"

**观察到的模式：**

| 场景类型 | Cosmos-Reason2-8B | Qwen3-VL-8B-Instruct |
|---------------|-------------------|----------------------|
| Side throw away | Correctly identifies motion away; no danger | Incorrectly flags approaching danger |
| Spin + throw toward | Correctly flags approach and potential risk | Mixed interpretation; risk downplayed |
| Lateral throw | Correctly identifies safe distance | Verbose but inconsistent distance framing |
| Toss upward | Flags possible approach during descent | Treats as generally safe; less decisive |

<details>
<summary><strong>模型原始输出</strong></summary>

**向侧面抛离：**

| 模型 | 输出 |
|-------|--------|
| Cosmos-Reason2-8B | "The hat lands on the desk near the person working at the computer. It is getting further away from you. There is no danger for the hat to hit you." |
| Qwen3-VL-8B-Instruct | "The hat is getting closer to you. It is falling downward and will likely land near your head or shoulders. There is a potential danger for the hat to hit you, so you should move your head away or duck to avoid it." |

**旋转后朝向机器人抛掷：**

| 模型 | 输出 |
|-------|--------|
| Cosmos-Reason2-8B | "The hat will land at the center of the frame, near the person. The hat is getting closer to me. Yes, there is a danger for the hat to hit me." |
| Qwen3-VL-8B-Instruct | "The hat is currently being tossed upward and will land on the floor in front of you, slightly to your left. It is getting closer to you as it descends, and there is a low risk of it hitting you since it will land on the floor before reaching you." |

**横向抛掷：**

| 模型 | 输出 |
|-------|--------|
| Cosmos-Reason2-8B | "The hat is moving upward and to the left, away from your position. It is getting further away from you, and there is no immediate danger of it hitting you." |
| Qwen3-VL-8B-Instruct | "The hat is currently in mid-air, moving upward and slightly forward from the woman's hand. Based on its trajectory, it will likely descend toward the floor or a nearby surface, not directly toward you. It is getting closer to the ceiling and then will begin to fall downward, so relative to your position, it is moving away from you as it ascends, then closer as it descends—but not in a direct path toward you. There is no immediate danger of the hat hitting you..." |

**向上抛起：**

| 模型 | 输出 |
|-------|--------|
| Cosmos-Reason2-8B | "The hat will land at the center of the frame, closer to the camera. It is moving closer to you, and there is a potential risk of it hitting you." |
| Qwen3-VL-8B-Instruct | "The hat is currently being thrown upward and is moving away from you as it ascends. It will reach a peak and then begin to descend back toward the ground. Since it's moving away from you at the moment, it is not getting closer. There is no immediate danger of the hat hitting you..." |

</details>

**解读：**

Cosmos-Reason2-8B 在以机器人为参照的空间与风险推理方面更加一致，而 Qwen3-VL-8B-Instruct 往往退回到描述性或叙事性解释，因此可能误判接近程度。

#### 3.2.2 动作预测的局限性

**提示：**

> "What is the person doing? Can you predict what is the next action this person likely to take?"

**观察：**

在一些情况下（例如旋转场景），Cosmos-Reason2-8B 会给出较为泛化的描述性解释，而不能稳定地预测接下来的投掷动作。Qwen3-VL-8B-Instruct 也表现出类似行为。

**含义：**

当提示明确以机器人为中心并与动作相关时，Cosmos-Reason2-8B 的优势最明显。对于中性的第三人称提示，它的行为会更接近基础模型。

### 3.3 多人空间推理

#### 场景

多个人出现在机器人周围不同深度的位置。

<img src="./assets/Test3-MultiPerson-Spatial-Reasoning.jpeg" width="400">

示例帧：从机器人视角看到多个处于不同距离的人

#### 提示

> "How many people are there in the view? Please describe the relative position of each person to me. Who is closest to me?"

#### 观察到的输出

| Aspect | Cosmos-Reason2-8B | Qwen3-VL-8B-Instruct |
|--------|-------------------|----------------------|
| Person count | Correct | Correct |
| Relative positions | Explicitly reasoned from robot view | Descriptive |
| Closest person | Explicitly identified | Correctly stated |

<details>
<summary><strong>模型原始输出</strong></summary>

**Cosmos-Reason2-8B:**

> *\<think\>*
> Okay, let's break this down. The user is asking how many people are in the video and who is closest to them. The video shows a man in a black T-shirt and glasses standing in front of the camera. He's the main subject, positioned centrally and closest to the camera. There's also a woman to his left, slightly behind him. She's wearing a dark top and light pants. Another man is in the background on the right, leaning against a desk. There's also a child on the desk near the man in the background. The camera is focused on the man in the black shirt, so he's definitely the closest. The others are behind or to the sides but not as close. So the answer should list the people and note the closest one.
> *\</think\>*
>
> There are four people in the view:
>
> 1. **The man in the black T-shirt and glasses** (closest to the camera, positioned centrally).
> 2. **The woman in a dark top and light pants** (to his left, slightly behind him).
> 3. **The man in the background on the right** (leaning against a desk, further away).
> 4. **The child on the desk** (near the man in the background).
>
> The **man in the black T-shirt** is closest to you.

**Qwen3-VL-8B-Instruct:**

> There are four people in the view.
>
> - The man in the black t-shirt is closest to me, standing in the center of the frame.
> - The woman in the black t-shirt is to my left, slightly behind the man.
> - The man in the blue t-shirt is to my right, standing behind a desk.
> - The child is seated at the desk to the right, behind the man in the blue t-shirt.
>
> The man in the black t-shirt is closest to me.

</details>

#### 解读

两个模型都能正确完成人数统计和基础定位。Cosmos-Reason2-8B 提供了更明确、基于机器人身体坐标系的推理，而基础模型给出的描述则更为平铺直叙。

### 3.4 社交情境感知（人与人握手）

#### 场景

机器人的视野中，两个人彼此握手。

<img src="./assets/Test4-Two-Person-Handshake.jpeg" width="400">

示例帧：从机器人视角观察到两个人正在握手

#### 提示

> "Does anyone in the view show interest to engage with me? Should I shake hands with anyone?"

#### 观察到的输出

| 模型 | 结果 |
|-------|---------|
| Cosmos-Reason2-8B | Correctly reasons the interaction is human-to-human and answers "No" |
| Qwen3-VL-8B-Instruct | Also answers "No", with a descriptive explanation |

<details>
<summary><strong>模型原始输出</strong></summary>

**Cosmos-Reason2-8B:**

> *\<think\>*
> Okay, let's break this down. The user is asking if they should shake hands with anyone in the camera view based on the shown interactions. The video shows a man in a black t-shirt and green pants who extends his hand to a woman entering the frame. They shake hands briefly, and then the woman moves out of view. The camera is static, so the user's perspective is from behind the camera.
>
> The key points here are: the man initiates the handshake with the woman, they complete it, and then she leaves. The user is behind the camera, observing this. The question is whether the user (behind the camera) should shake hands with anyone. Since the handshake happens between the man and the woman, and the user isn't shown as participating in that interaction, it seems like the user isn't part of the handshake. The man and woman are the ones interacting, and the user is just observing. So, the answer would be no, because the user isn't involved in the handshake. The interaction is between the man and the woman, not the user. The user's presence is passive, just watching. Therefore, there's no need for the user to shake hands with anyone in the scene.
> *\</think\>*
>
> **No**

**Qwen3-VL-8B-Instruct:**

> No one in the view appears to show interest in engaging with you (behind the camera). There is no indication that anyone is aware of or looking toward you. Additionally, there is no one in the frame who you should shake hands with, as no one is approaching or initiating interaction with you. The people in the scene are either engaged in their own activities or interacting with each other, but not with you.

</details>

#### 解读

两个模型都得出了正确结论。Cosmos-Reason2-8B 明确推理了社交参与边界，进一步强化了恰当的不介入行为。

## 4. 提示设计指导

基于观察到的行为：

- **显式的第一视角框定至关重要**（“the camera view is my view”）
- **相对的、以机器人为中心的问题优于纯描述性问题**（“closer to me”“danger to me”）
- **面向动作的提示更能凸显 Cosmos 的优势**
- **中性的第三人称提示会减弱** 与基础模型之间的差异

## 5. 对比总结

| 维度 | Cosmos-Reason2-8B | Qwen3-VL-8B-Instruct |
|-----------|-------------------|----------------------|
| 面向机器人的人类意图 | 强 | 弱 |
| 相对机器人的物体运动 | 更一致 | 不一致 |
| 碰撞风险评估 | 保守且有依据 | 经常高估或低估 |
| 多人空间推理 | 明确 | 描述性 |
| 社交边界感知 | 明确 | 正确但隐含 |

## 6. 关键结论

根据已评估的测试结果：

- **Cosmos-Reason2-8B 在从具身机器人视角进行推理时展现出明显优势**，尤其体现在意图识别、物理风险评估和社交参与决策方面。
- **对于中性或第三人称提示**，其行为会向基础模型收敛，这也凸显了提示与具身设定对齐的重要性。

这些结果表明，Cosmos-Reason2-8B 是社交机器人和物理 AI 系统的强大基础，在这些系统中，物理规律约束和动作相关性至关重要。

### 局限性与未来工作

本次评估在受限的时间范围内完成，这限制了基准设计的覆盖范围。因此，当前测试集更适合作为初步探索，而不是完整全面的基准。未来工作应关注以下方面：

- **扩展测试覆盖范围** —— 增加测试场景的数量和多样性，以提升统计稳健性
- **系统化示例选择** —— 对场景选择采用更严格的标准，确保不同交互类型之间具有均衡代表性
- **补充能力测试** —— 纳入第 1 节中提到的物体交接和情绪状态感知测试

---

## 文档信息

**发布日期：** 2026 年 1 月 1 日

### 引用

如果你使用了本配方或参考了这项工作，请按如下方式引用：

```bibtex
@misc{cosmos_cookbook_intbot_2026,
  title={Egocentric Social and Physical Reasoning with Cosmos-Reason2-8B},
  author={Zhang, Tingyu and Yang, Sharon},
  organization={IntBot Inc.},
  year={2026},
  month={January},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/reason2/intbot_showcase/inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的正文引用：**

> Tingyu Zhang, & Sharon Yang (2026). Egocentric Social and Physical Reasoning with Cosmos-Reason2-8B. In *NVIDIA Cosmos Cookbook*. IntBot Inc. Accessible at <https://nvidia-cosmos.github.io/cosmos-cookbook/recipes/inference/reason2/intbot_showcase/inference.html>
