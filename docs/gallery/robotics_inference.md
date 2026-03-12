# 机器人领域自适应示例集

> **作者：** [Raju Wagwani](https://www.linkedin.com/in/raju-wagwani-a4746027/) • [Jathavan Sriram](https://www.linkedin.com/in/jathavansriram) • [Richard Yarlett](https://www.linkedin.com/in/richardyarlett/) • [Joshua Bapst](https://www.linkedin.com/in/joshbapst/) • [Jinwei Gu](https://www.linkedin.com/in/jinweigu/)

> **机构：** NVIDIA

## 概述

本页展示了 Cosmos Transfer 2.5 在机器人应用中的结果。这些示例演示了厨房环境下机器人操作任务的 sim-to-real 迁移，展示了如何将合成仿真视频转换为具有不同材质、光照和环境条件的照片级真实场景。这些结果可用于机器人训练与验证中的领域自适应和数据增强。

**使用场景**：机器人工程师可以使用这些技术，从单次仿真中生成多样化训练数据，在无需重新运行高成本仿真或采集真实世界数据的情况下，创建不同的厨房风格、材质和光照条件变体。

## 示例 1：仅使用 Edge Control 实现环境变化

该示例演示了如何使用 **edge control** 将合成机器人仿真视频转换为具有不同厨房风格和材质的照片级真实场景。该控制方式在允许视觉外观根据文本提示显著变化的同时，保留了机器人与场景原始的结构、运动和几何信息。

- **Edge control**：保留仿真中的物体结构与布局、机器人姿态以及相机运动，同时根据提示词改变视觉外观（材质、光照、颜色）。
- **为何仅使用 edge**：在改变环境外观风格的同时，精确保留仿真中的机器人运动和物体位置。

有关控制模态的详细说明，请参阅[控制模态总览](../core_concepts/control_modalities/overview.md)。

<style>
.carousel {
  position: relative;
  margin-top: 1rem;
  overflow: visible;
}

.carousel-track {
  position: relative;
}

.carousel-slide {
  display: none;
  flex-direction: column;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid var(--md-default-fg-color--lightest, #e2e8f0);
  background: var(--md-default-bg-color, #fff);
  box-shadow: 0 10px 34px rgba(0, 0, 0, 0.06);
  border-radius: 0px;
}

.carousel-slide.is-active {
  display: flex;
}

.media-wrap {
  position: relative;
  overflow: visible;
  background: #000;
  margin-bottom: 0.5rem;
  border-radius: 0px;
}

.media-wrap video {
  width: 100%;
  display: block;
  border-radius: 0px;
}

.carousel-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 36px;
  height: 36px;
  border: none;
  background: var(--md-accent-fg-color, #76b900);
  color: #fff;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.16);
  opacity: 1;
  z-index: 3;
  font-size: 18px;
  line-height: 1;
  font-weight: 700;
  border-radius: 50%;
}

.carousel-btn:hover {
  opacity: 1;
}

.carousel-btn.prev {
  left: -18px;
}

.carousel-btn.next {
  right: -18px;
}

.text-stack {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.text-block .label {
  font-weight: 700;
  text-transform: uppercase;
  font-size: 0.75rem;
  letter-spacing: 0.04em;
  color: var(--md-default-fg-color--light, #5b6472);
}

.text-block .preview-text {
  display: block;
}

.text-block .full-text {
  display: none;
  margin-top: 0.25rem;
}

.carousel-slide.expanded .preview-text {
  display: none;
}

.carousel-slide.expanded .full-text {
  display: block;
}

.see-more {
  align-self: flex-start;
  padding: 0.4rem 0.8rem;
  border: 1px solid var(--md-accent-fg-color, #76b900);
  background: transparent;
  cursor: pointer;
  font-weight: 600;
  color: var(--md-accent-fg-color, #76b900);
  border-radius: 0px;
}

/*Format parameters JSON with line breaks*/
.text-block .full-text {
  white-space: pre-wrap;
  font-family: var(--md-code-font, monospace);
  font-size: 0.9em;
}

.text-block .preview-text {
  font-family: var(--md-code-font, monospace);
  font-size: 0.9em;
}
</style>

### 场景 1a：厨房炉灶 - 烹饪任务

该场景展示了一台人形机器人在炉灶前执行烹饪任务。示例演示了如何从同一段仿真中生成不同的厨房橱柜风格（白色、红色、木色）以及不同的机器人材质（塑料、金属、金色）。

#### 输入视频

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_stove_input.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "seed": 1,</span>
          <span class="full-text">{
    "seed": 1,
    "prompt_path": "assets/prompt_robot.json",
    "output_dir": "outputs/robot",
    "video_path": "assets/kitchen_stove_input.mp4",
    "guidance": 7,
    "edge": {
        "control_weight": 1
    }
}</span>
        </div>
        <button class="see-more" type="button">展开完整参数</button>
      </div>
    </article>
  </div>
</div>

#### 示例

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_stove_white.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all highly polished bright white panels with chrome accents and pulls. The kitchen counters are stainless steel. The kitchen walls and backsplash are all white subway tile. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. Standing in the kitchen is a humanoid robot. The robot is made of orange polished plastic panels with chrome accents. The camera is fixed and steady. The robot is at a kitchen stainless steel stove picking up a glass cooking pot lid with his left hand and lifting it in the air. The robot is picking up two tomatoes with his right hand and putting them inside the stainless steel pot. There is steam coming out of the pot.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_stove_red.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all highly polished bright red panels with stainless steel accents and pulls. The kitchen counters are stainless steel. The kitchen walls and backsplash are all white subway tile. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. Standing in the kitchen is a humanoid robot. The robot is made of white polished panels with black accents. The camera is fixed and steady. The robot is at a kitchen stainless steel stove picking up a red cooking pot lid with his left hand and lifting it in the air. The robot is picking up two tomatoes with his right hand and putting them inside the red pot. There is steam coming out of the pot.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_stove_light_wood.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all light wood, with stainless steel accents and pulls. The kitchen counters, kitchen walls and backsplash are all expensive black veined marble. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. Standing in the kitchen is a humanoid robot. The robot is made of stainless steel polished panels with chrome accents. The camera is fixed and steady. The robot is at a kitchen stainless steel stove picking up a stainless steel cooking pot lid with his left hand and lifting it in the air. The robot is picking up two tomatoes with his right hand and putting them inside the stainless steel pot. There is steam coming out of the pot.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_stove_dark_wood.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all dark wood, with chrome accents and pulls. The kitchen counters, kitchen walls and backsplash are all expensive beige veined marble. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. Standing in the kitchen is a humanoid robot. The robot is made of gold polished reflective panels with shiny black accents. The camera is fixed and steady. The robot is at a kitchen gold stove picking up a gold cooking pot lid with his left hand and lifting it in the air. The robot is picking up two tomatoes with his right hand and putting them inside the gold pot. There is steam coming out of the pot.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_stove.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">The video captures a stunning, photorealistic scene with remarkable attention to detail, giving it a lifelike appearance that is almost indistinguishable from reality. It appears to be from a high-budget 4K movie, showcasing ultra-high-definition quality with impeccable resolution.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

  </div>
</div>

### 场景 1b：厨房岛台 - 物体操作

该场景展示了机器人在厨房岛台上执行精细物体操作，完成拾取和放置动作。示例演示了材质变化（不同水果/物体）如何与厨房风格变化相协调。

#### 输入视频

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_oranges_input.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "seed": 1,</span>
          <span class="full-text">{
    "seed": 1,
    "prompt_path": "assets/prompt_robot.json",
    "output_dir": "outputs/robot",
    "video_path": "assets/kitchen_oranges_input.mp4",
    "guidance": 7,
    "edge": {
        "control_weight": 1
    }
}</span>
        </div>
        <button class="see-more" type="button">展开完整参数</button>
      </div>
    </article>
  </div>
</div>

#### 示例

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_oranges_white.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all highly polished bright white panels with chrome accents and pulls. The kitchen counters are stainless steel. The kitchen walls and backsplash are all white subway tile. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with highly polished bright white panels cabinets and an stainless steel countertop. In the middle of the island counter is a large glass bowl of oranges. Standing in the kitchen is a humanoid robot. The robot is made of orange polished plastic panels with chrome accents. The camera is fixed and steady. The robot is picking up two oranges from either side of a small glass plate, and placing them on the plate.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_oranges_red.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all highly polished bright red panels with stainless steel accents and pulls. The kitchen counters are stainless steel. The kitchen walls and backsplash are all white subway tile. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with highly polished bright red panels cabinets and an stainless steel countertop. In the middle of the island counter is a large white bowl of eggs. Standing in the kitchen is a humanoid robot. The robot is made of white polished panels with black accents. The camera is fixed and steady. The robot is picking up two eggs from either side of a small white plate, and placing them on the plate.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_oranges_light_wood.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all light wood, with stainless steel accents and pulls. The kitchen counters, kitchen walls and backsplash are all expensive black veined marble. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with light wood cabinets and an expensive black veined marble countertop. In the middle of the island counter is a large white bowl of lemons. Standing in the kitchen is a humanoid robot. The robot is made of stainless steel polished panels with chrome accents. The camera is fixed and steady. The robot is picking up two lemons from either side of a small white plate, and placing them on the plate.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_oranges_dark_wood.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all dark wood, with chrome accents and pulls. The kitchen counters, kitchen walls and backsplash are all expensive beige veined marble. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with dark wood cabinets and an expensive beige veined marble countertop. In the middle of the island counter is a large white bowl of apples. Standing in the kitchen is a humanoid robot. The robot is made of gold polished reflective panels with shiny black accents. The camera is fixed and steady. The robot is picking up two apples from either side of a small white plate, and placing them on the plate.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_oranges.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">The video captures a stunning, photorealistic scene with remarkable attention to detail, giving it a lifelike appearance that is almost indistinguishable from reality. It appears to be from a high-budget 4K movie, showcasing ultra-high-definition quality with impeccable resolution.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

  </div>
</div>

### 场景 1c：厨房冰箱 - 家电交互

该场景演示了机器人与家电的交互，展示机器人打开冰箱的过程。示例在改变厨房美学风格的同时，保持了光照动态（冰箱内部灯光）。

#### 输入视频

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_fridge_input.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "seed": 1,</span>
          <span class="full-text">{
    "seed": 1,
    "prompt_path": "assets/prompt_robot.json",
    "output_dir": "outputs/robot",
    "video_path": "assets/kitchen_fridge_input.mp4",
    "guidance": 7,
    "edge": {
        "control_weight": 1
    }
}</span>
        </div>
        <button class="see-more" type="button">展开完整参数</button>
      </div>
    </article>
  </div>
</div>

#### 示例

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_fridge_white.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all highly polished bright white panels with chrome accents and pulls. The kitchen counters are stainless steel. The kitchen walls and backsplash are all white subway tile. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with highly polished bright white panels cabinets and an stainless steel countertop. In the middle of the island counter is a large glass bowl of oranges. Standing in the kitchen is a humanoid robot. The robot is made of orange polished plastic panels with chrome accents. The camera is fixed and steady. The robot is opening the fridge with his right hand and looking inside. The fridge light turns on and it very bright, showing the inside of the fridge filled with food and drink.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_fridge_red.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all highly polished bright red panels with stainless steel accents and pulls. The kitchen counters are stainless steel. The kitchen walls and backsplash are all white subway tile. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with highly polished bright red panels cabinets and an stainless steel countertop. In the middle of the island counter is a large white bowl of eggs. Standing in the kitchen is a humanoid robot. The robot is made of white polished panels with black accents. The camera is fixed and steady. The robot is opening the fridge with his right hand and looking inside. The fridge light turns on and it very bright, showing the inside of the fridge filled with food and drink.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_fridge_light_wood.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all light wood, with stainless steel accents and pulls. The kitchen counters, kitchen walls and backsplash are all expensive black veined marble. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with light wood cabinets and an expensive black veined marble countertop. In the middle of the island counter is a large white bowl of lemons. Standing in the kitchen is a humanoid robot. The robot is made of stainless steel polished panels with chrome accents. The camera is fixed and steady. The robot is opening the fridge with his right hand and looking inside. The fridge light turns on and it very bright, showing the inside of the fridge filled with food and drink.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_fridge_dark_wood.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">This scene depicts a photo realistic luxury kitchen with high end professional finishes and lighting. ALL The kitchen cabinets are all dark wood, with chrome accents and pulls. The kitchen counters, kitchen walls and backsplash are all expensive beige veined marble. The kitchen contains an expensive double door stainless steel refrigerator, a stainless steel microwave, a stainless steel oven, a stainless steel coffee machine, a stainless steel toaster, a stainless steel stove top, a stainless steel sink, and stainless steel pots. In the center of the room is a kitchen island. This is also finished with dark wood cabinets and an expensive beige veined marble countertop. In the middle of the island counter is a large white bowl of apples. Standing in the kitchen is a humanoid robot. The robot is made of gold polished reflective panels with shiny black accents. The camera is fixed and steady. The robot is opening the fridge with his right hand and looking inside. The fridge light turns on and it very bright, showing the inside of the fridge filled with food and drink.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen_fridge.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">The video captures a stunning, photorealistic scene with remarkable attention to detail, giving it a lifelike appearance that is almost indistinguishable from reality. It appears to be from a high-budget 4K movie, showcasing ultra-high-definition quality with impeccable resolution.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">seed: 1, guidance: 7, edge: 1.0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

  </div>
</div>

## 示例 2：结合自定义控制视频的多控制

这些示例展示了更高级的用法：您可以在输入视频之外，额外提供**预先计算好的自定义控制视频**（depth、edge、segmentation）。多控制可让您对转换的不同方面进行更细粒度的控制：

- **depth**：控制 3D 空间关系和透视效果
- **edge**：保持结构边界和物体形状
- **seg**：支持语义层面的变化和物体替换
- **vis**：保留光照和相机属性（本示例中设为 0）

**何时使用多控制**：当您需要通过预生成并精调特定控制信号来精确控制转换过程时，可采用此方法，尤其适用于复杂场景编辑或仅靠 edge control 不足以满足需求的情况。

### 场景 2a

### 输入与控制视频

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen2_cg.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">输入视频</span>
          <span class="full-text">输入视频</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "seed": 2000,</span>
          <span class="full-text">{
    "seed": 2000,
    "prompt_path": "assets/prompt_kitchen2.json",
    "video_path": "assets/kitchen2_cg.mp4",
    "guidance": 2,
    "depth": {
        "control_path": "assets/kitchen2_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/kitchen2_edge.mp4",
        "control_weight": 0.2
    },
    "seg": {
        "control_path": "assets/kitchen2_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
        "control_weight": 0
    }
}</span>
        </div>
        <button class="see-more" type="button">展开完整参数</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen2_depth.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">深度控制</span>
          <span class="full-text">深度控制</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "seed": 2000,</span>
          <span class="full-text">{
    "seed": 2000,
    "prompt_path": "assets/prompt_kitchen2.json",
    "video_path": "assets/kitchen2_cg.mp4",
    "guidance": 2,
    "depth": {
        "control_path": "assets/kitchen2_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/kitchen2_edge.mp4",
        "control_weight": 0.2
    },
    "seg": {
        "control_path": "assets/kitchen2_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
        "control_weight": 0
    }

}</span>

</div>
<button class="see-more" type="button">展开完整参数</button>
</div>
</article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen2_edge.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">边缘控制</span>
          <span class="full-text">边缘控制</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "seed": 2000,</span>
          <span class="full-text">{
    "seed": 2000,
    "prompt_path": "assets/prompt_kitchen2.json",
    "video_path": "assets/kitchen2_cg.mp4",
    "guidance": 2,
    "depth": {
        "control_path": "assets/kitchen2_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/kitchen2_edge.mp4",
        "control_weight": 0.2
    },
    "seg": {
        "control_path": "assets/kitchen2_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
        "control_weight": 0
    }

}</span>

</div>
<button class="see-more" type="button">展开完整参数</button>
</div>
</article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/kitchen2_seg.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">分割控制</span>
          <span class="full-text">分割控制</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "seed": 2000,</span>
          <span class="full-text">{
    "seed": 2000,
    "prompt_path": "assets/prompt_kitchen2.json",
    "video_path": "assets/kitchen2_cg.mp4",
    "guidance": 2,
    "depth": {
        "control_path": "assets/kitchen2_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/kitchen2_edge.mp4",
        "control_weight": 0.2
    },
    "seg": {
        "control_path": "assets/kitchen2_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
        "control_weight": 0
    }

}</span>

</div>
<button class="see-more" type="button">展开完整参数</button>
</div>
</article>

  </div>
</div>

### 输出视频

<div class="media-wrap">
  <video autoplay loop muted playsinline>
    <source src="assets/kitchen2_output.mp4" type="video/mp4">
    您的浏览器不支持视频标签。
  </video>
</div>

### 场景 2b

### 输入与控制视频

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_input.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">输入视频</span>
          <span class="full-text">输入视频</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube.</span>
          <span class="full-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "name": "robot_multicontrol",</span>
          <span class="full-text">{
    "name": "robot_multicontrol",
    "video_path": "assets/robotic_arm_input.mp4",
    "guidance": 3,
    "depth": {
        "control_path": "assets/robotic_arm_input_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/robotic_arm_input_edge.mp4",
        "control_weight": 1
    },
    "seg": {
        "control_path": "assets/robotic_arm_input_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
       "control_path": "assets/robotic_arm_input_vis.mp4",
        "control_weight": 0
    },
    "prompt": "The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light."
}</span>
        </div>
        <button class="see-more" type="button">展开完整参数</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_input_depth.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">深度控制</span>
          <span class="full-text">深度控制</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube.</span>
          <span class="full-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "name": "robot_multicontrol",</span>
          <span class="full-text">{
    "name": "robot_multicontrol",
    "video_path": "assets/robotic_arm_input.mp4",
    "guidance": 3,
    "depth": {
        "control_path": "assets/robotic_arm_input_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/robotic_arm_input_edge.mp4",
        "control_weight": 1
    },
    "seg": {
        "control_path": "assets/robotic_arm_input_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
       "control_path": "assets/robotic_arm_input_vis.mp4",
        "control_weight": 0
    },
    "prompt": "The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light."

}</span>

</div>
<button class="see-more" type="button">展开完整参数</button>
</div>
</article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_input_edge.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">边缘控制</span>
          <span class="full-text">边缘控制</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube.</span>
          <span class="full-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "name": "robot_multicontrol",</span>
          <span class="full-text">{
    "name": "robot_multicontrol",
    "video_path": "assets/robotic_arm_input.mp4",
    "guidance": 3,
    "depth": {
        "control_path": "assets/robotic_arm_input_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/robotic_arm_input_edge.mp4",
        "control_weight": 1
    },
    "seg": {
        "control_path": "assets/robotic_arm_input_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
       "control_path": "assets/robotic_arm_input_vis.mp4",
        "control_weight": 0
    },
    "prompt": "The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light."

}</span>

</div>
<button class="see-more" type="button">展开完整参数</button>
</div>
</article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_input_seg.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">分割控制</span>
          <span class="full-text">分割控制</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube.</span>
          <span class="full-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "name": "robot_multicontrol",</span>
          <span class="full-text">{
    "name": "robot_multicontrol",
    "video_path": "assets/robotic_arm_input.mp4",
    "guidance": 3,
    "depth": {
        "control_path": "assets/robotic_arm_input_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/robotic_arm_input_edge.mp4",
        "control_weight": 1
    },
    "seg": {
        "control_path": "assets/robotic_arm_input_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
       "control_path": "assets/robotic_arm_input_vis.mp4",
        "control_weight": 0
    },
    "prompt": "The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light."

}</span>

</div>
<button class="see-more" type="button">展开完整参数</button>
</div>
</article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_input_vis.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">视频类型</div>
          <span class="preview-text">Vis 控制</span>
          <span class="full-text">Vis 控制</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube.</span>
          <span class="full-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{ "name": "robot_multicontrol",</span>
          <span class="full-text">{
    "name": "robot_multicontrol",
    "video_path": "assets/robotic_arm_input.mp4",
    "guidance": 3,
    "depth": {
        "control_path": "assets/robotic_arm_input_depth.mp4",
        "control_weight": 0.6
    },
    "edge": {
        "control_path": "assets/robotic_arm_input_edge.mp4",
        "control_weight": 1
    },
    "seg": {
        "control_path": "assets/robotic_arm_input_seg.mp4",
        "control_weight": 0.4
    },
    "vis": {
       "control_path": "assets/robotic_arm_input_vis.mp4",
        "control_weight": 0
    },
    "prompt": "The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light."

}</span>

</div>
<button class="see-more" type="button">展开完整参数</button>
</div>
</article>

  </div>
</div>

#### 示例

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_1.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">The video features two robotic arms with brushed matte black bodies, and contrasting black joints, manipulating a small red glass cube. They are positioned on a plastic table, with minimalistic office in the background, illuminated by artificial white light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, depth: 0.6, edge: 1.0, seg: 0.4, vis: 0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">The video features two robotic arms with brushed bronze bodies, and contrasting yellow joints, manipulating a small purple plastic cube. They are positioned on a granite table, with urban rooftop in the background, illuminated by natural light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, depth: 0.6, edge: 1.0, seg: 0.4, vis: 0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_3.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">The video features two robotic arms with matte black bodies, and contrasting blue joints, manipulating a small white plastic cube. They are positioned on a marble table, with industrial warehouse in the background, illuminated by colored ambient light (blue).</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, depth: 0.6, edge: 1.0, seg: 0.4, vis: 0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robotic_arm_4.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">The video features two robotic arms with matte white bodies, and contrasting black joints, manipulating a small green glass cube. They are positioned on a marble table, with closed room in the background, illuminated by artificial white light.</span>
        </div>
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, depth: 0.6, edge: 1.0, seg: 0.4, vis: 0</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>

  </div>
</div>

<script>
document.addEventListener("DOMContentLoaded", () => {
  const firstSentence = (text) => {
    const trimmed = text.trim();
    const match = trimmed.match(/.*?[.!?](\s|$)/);
    return match ? match[0].trim() : trimmed;
  };

  document.querySelectorAll(".carousel").forEach((carousel) => {
    const slides = Array.from(carousel.querySelectorAll(".carousel-slide"));
    if (!slides.length) return;

    slides.forEach((slide) => {
      slide.classList.remove("expanded");
      slide.querySelectorAll(".text-block").forEach((block) => {
        const full = block.querySelector(".full-text");
        const preview = block.querySelector(".preview-text");
        if (full && preview) {
          // Only set preview if it's empty (preserve manually set previews like "{ "seed": 1,")
          if (!preview.textContent.trim()) {
            preview.textContent = firstSentence(full.textContent || "");
          }
        }
      });
      const toggle = slide.querySelector(".see-more");
      if (toggle) {
        const originalText = toggle.textContent.trim();
        const isParameters = originalText.includes("参数");
        toggle.addEventListener("click", () => {
          slide.classList.toggle("expanded");
          const expanded = slide.classList.contains("expanded");
          if (isParameters) {
            toggle.textContent = expanded ? "收起完整参数" : "展开完整参数";
          } else {
            toggle.textContent = expanded ? "收起完整提示词" : "展开完整提示词";
          }
        });
      }
    });

    let index = slides.findIndex((s) => s.classList.contains("is-active"));
    if (index < 0) {
      index = 0;
      slides[0].classList.add("is-active");
    }

    const intervalMs = parseInt(carousel.dataset.interval || "5000", 10);
    const show = (nextIndex) => {
      slides[index].classList.remove("is-active", "expanded");
      const priorToggle = slides[index].querySelector(".see-more");
      if (priorToggle) {
        const originalText = priorToggle.textContent.trim();
        const isParameters = originalText.includes("参数");
        priorToggle.textContent = isParameters ? "展开完整参数" : "展开完整提示词";
      }
      index = (nextIndex + slides.length) % slides.length;
      slides[index].classList.add("is-active");
    };

    const next = () => show(index + 1);
    const prev = () => show(index - 1);

    let timer = slides.length > 1 ? setInterval(next, intervalMs) : null;
    const resetTimer = () => {
      if (!timer) return;
      clearInterval(timer);
      timer = setInterval(next, intervalMs);
    };

    carousel.querySelectorAll(".carousel-btn.next").forEach((btn) => {
      btn.addEventListener("click", () => {
        next();
        resetTimer();
      });
    });
    carousel.querySelectorAll(".carousel-btn.prev").forEach((btn) => {
      btn.addEventListener("click", () => {
        prev();
        resetTimer();
      });
    });
  });
});
</script>

## 质量提升：Transfer 2.5 对比 Transfer 1

与 Cosmos Transfer 1 相比，Cosmos Transfer 2.5 在**视频质量**和**推理速度**两方面都实现了显著提升。下方示例展示了并排对比效果，每段视频都会在 Transfer 1 结果与 Transfer 2.5 结果之间切换，以呈现最新版本带来的质量改进。

### 示例

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robot1_t1_t2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/robot2_t1_t2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
    </article>

  </div>
</div>

---

## 文档信息

**发布日期：**2025 年 11 月 12 日

### 引用

如果您使用了本内容或引用了这项工作，请按以下方式引用：

```bibtex
@misc{cosmos_cookbook_robotics_gallery_2025,
  title={Robotics Domain Adaptation Gallery},
  author={Wagwani, Raju and Sriram, Jathavan and Yarlett, Richard and Bapst, Joshua and Gu, Jinwei},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/gallery/robotics_inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Raju Wagwani、Jathavan Sriram、Richard Yarlett、Joshua Bapst 和 Jinwei Gu（2025）。《机器人领域自适应示例集》。载于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/gallery/robotics_inference.html>
