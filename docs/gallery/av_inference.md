# 自动驾驶汽车领域自适应示例集

> **作者：** [Raju Wagwani](https://www.linkedin.com/in/raju-wagwani-a4746027/) • [Nikolay Matveiev](https://www.linkedin.com/) • [Joshua Bapst](https://www.linkedin.com/in/joshbapst/) • [Jinwei Gu](https://www.linkedin.com/in/jinweigu/)

> **机构：** NVIDIA

## 概述

本页展示了一组使用 Cosmos Transfer 2.5 为自动驾驶汽车（AV）应用生成的结果。这些示例演示了如何在不同天气、光照和一天中不同时段等多种环境条件下，对真实世界或基于仿真的驾驶视频进行转换。这些结果旨在为探索如何利用该模型进行领域自适应和自动驾驶场景合成数据增强的用户提供灵感。

## 驾驶场景 1

- **多控制**：模型会使用不同的控制信号，并为每种控制分配不同的权重来生成输出。这让用户能够更精细地控制最终结果的呈现方式。
  - **depth**：保持 3D 真实感和空间一致性。
  - **edge**：保留原始结构、形状和布局。
  - **seg**：支持结构变化和语义替换。
  - **vis**：保留背景、光照和整体视觉外观。

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
}

.carousel-slide.is-active {
  display: flex;
}

.media-wrap {
  position: relative;
  overflow: visible;
  background: #000;
  margin-bottom: 0.5rem;
}

.media-wrap video {
  width: 100%;
  display: block;
}

/*Override Material theme default border-radius*/
.carousel-slide,
.media-wrap,
.media-wrap video,
.see-more,
.masonry-card {
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
  font-family: var(--md-code-font, monospace);
  font-size: 0.9em;
}

.text-block .full-text {
  display: none;
  margin-top: 0.25rem;
  white-space: pre-wrap;
  font-family: var(--md-code-font, monospace);
  font-size: 0.9em;
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
}
</style>

### 输入视频

该场景展示了一段以行车记录仪视角拍摄的驾驶视频。下方示例演示了如何在保持驾驶场景结构与运动不变的前提下，从同一输入视频生成不同的环境条件（天气、光照、时间段）。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/av_car_input.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text">{
  "seed": 5000,
}</span>
          <span class="full-text">{
    // Update the parameter values for control weights, seed, guidance in below json file
    "seed": 5000,
    "prompt_path": "assets/prompt_av.json",           // Update the prompt in the json file accordingly
    "video_path": "assets/av_car_input.mp4",
    "guidance": 3,
    "depth": {
        "control_weight": 0.4
    },
    "edge": {
        "control_weight": 0.1
    },
    "seg": {
        "control_weight": 0.5
    },
    "vis": {
        "control_weight": 0.1
    }
}</span>
        </div>
        <button class="see-more" type="button">展开完整参数</button>
      </div>
    </article>
  </div>
</div>

### 示例

<style>
.masonry-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.1rem;
}

@media (max-width: 720px) {
  .masonry-grid {
    grid-template-columns: 1fr;
  }
}

.masonry-card {
  position: relative;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  background: #000;
}

.masonry-card video {
  width: 100%;
  display: block;
}

.masonry-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.88);
  color: #fff;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  gap: 0.5rem;
  opacity: 0;
  transition: opacity 150ms ease;
  overflow-y: auto;
}

.masonry-card:hover .masonry-overlay {
  opacity: 1;
}

.masonry-overlay .label {
  font-weight: 700;
  text-transform: uppercase;
  font-size: 0.75rem;
  letter-spacing: 0.04em;
  color: #e2e8f0;
}

.masonry-overlay .prompt,
.masonry-overlay .params {
  font-size: 0.9rem;
  line-height: 1.4;
  white-space: normal;
}

.masonry-card:hover .masonry-overlay .prompt,
.masonry-card:hover .masonry-overlay .params {
  font-size: 0.75rem;
}

</style>

<div class="masonry-grid">
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_1.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">The video is a driving scene through a modern urban environment, likely captured from a dashcam or a similar fixed camera setup inside a vehicle. The scene unfolds on a wide, multi-lane road flanked by tall, modern buildings with glass facades. The road is relatively empty, with only a few cars visible, including a black car directly ahead of the camera, maintaining a steady pace. The camera remains static, providing a consistent view of the road and surroundings as the vehicle moves forward.On the left side of the road, there are several trees lining the sidewalk, providing a touch of greenery amidst the urban setting. Pedestrians are visible on the sidewalks, some walking leisurely, while others stand near the buildings. The buildings are a mix of architectural styles, with some featuring large glass windows and others having more traditional concrete exteriors. A few commercial signs and logos are visible on the buildings, indicating the presence of businesses and offices.Traffic cones are placed on the road ahead, suggesting some form of roadwork or lane closure, guiding the vehicles to merge or change lanes. The road markings are clear, with white arrows indicating the direction of travel. Throughout the video, the vehicle maintains a steady speed, and the camera captures the gradual approach towards the intersection, where the road splits into different directions. The overall atmosphere is calm and orderly, typical of a city during non-peak hours.  heavy rain, wet road with puddles</div>
      <div class="label">参数</div>
      <div class="params">seed: 4000, guidance: 3, depth: 0.5, edge: 0.1, seg: 0.35, vis: 0.0</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_2.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">The video is a driving scene through a modern urban environment, likely captured from a dashcam or a similar fixed camera setup inside a vehicle. The scene unfolds on a wide, multi-lane road flanked by tall, modern buildings with glass facades. The road is relatively empty, with only a few cars visible, including a black car directly ahead of the camera, maintaining a steady pace. The camera remains static, providing a consistent view of the road and surroundings as the vehicle moves forward.On the left side of the road, there are several trees lining the sidewalk, providing a touch of greenery amidst the urban setting. Pedestrians are visible on the sidewalks, some walking leisurely, while others stand near the buildings. The buildings are a mix of architectural styles, with some featuring large glass windows and others having more traditional concrete exteriors. A few commercial signs and logos are visible on the buildings, indicating the presence of businesses and offices.Traffic cones are placed on the road ahead, suggesting some form of roadwork or lane closure, guiding the vehicles to merge or change lanes. The road markings are clear, with white arrows indicating the direction of travel. Throughout the video, the vehicle maintains a steady speed, and the camera captures the gradual approach towards the intersection, where the road splits into different directions. The overall atmosphere is calm and orderly, typical of a city during non-peak hours.  night time, bright street lamps and colorful neon lights on buildings</div>
      <div class="label">参数</div>
      <div class="params">seed: 4000, guidance: 7, depth: 0.5, edge: 0.1, seg: 0.35, vis: 0.0</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_3.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, sunset with beautiful clouds</div>
      <div class="label">参数</div>
      <div class="params">seed: 5000, guidance: 3, depth: 0.35, edge: 0.1, seg: 0.35, vis: 0.0</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_4.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, heavy rain, wet road with puddles</div>
      <div class="label">参数</div>
      <div class="params">seed: 5000, guidance: 3, depth: 0.35, edge: 0.1, seg: 0.35, vis: 0.0</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_5.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, winter with heavy snow storm, trees and sidewalks covered in snow</div>
      <div class="label">参数</div>
      <div class="params">seed: 8001, guidance: 3, depth: 0.35, edge: 0.15, seg: 0.35, vis: 0.15</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_6.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, heavy rain, wet road with puddles</div>
      <div class="label">参数</div>
      <div class="params">seed: 5000, guidance: 6, depth: 0.6, edge: 0.1, seg: 0.4, vis: 0.1</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_7.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, sunset with beautiful clouds</div>
      <div class="label">参数</div>
      <div class="params">seed: 7000, guidance: 3, depth: 0.35, edge: 0.1, seg: 0.35, vis: 0.05</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_8.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, winter with heavy snow storm, trees and sidewalks covered in snow</div>
      <div class="label">参数</div>
      <div class="params">seed: 7001, guidance: 3, depth: 0.35, edge: 0.1, seg: 0.35, vis: 0.05</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_9.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, twilight or early morning, partly cloudy</div>
      <div class="label">参数</div>
      <div class="params">seed: 5000, guidance: 3, depth: 0.4, edge: 0.1, seg: 0.5, vis: 0.1</div>
    </div>
  </div>

  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="assets/av_car_output_10.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">输入提示词</div>
      <div class="prompt">Dashcam video, driving through a modern urban environment, night time, bright street lamps lighting up the fog</div>
      <div class="label">参数</div>
      <div class="params">seed: 5000, guidance: 3, depth: 0.4, edge: 0.1, seg: 0.5, vis: 0.1</div>
    </div>
  </div>
</div>

## 质量提升：Transfer 2.5 对比 Transfer 1

与 Cosmos Transfer 1 相比，Cosmos Transfer 2.5 在**视频质量**和**推理速度**两方面都带来了显著提升。下方示例展示了并排对比效果，每段视频都会在 Transfer 1 结果与 Transfer 2.5 结果之间切换，以呈现最新版本实现的质量提升。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/av1_t1_t2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">对比</div>
          <span class="preview-text">示例 A：并排对比展示了从 Transfer 1 到 Transfer 2.5 的质量提升。</span>
          <span class="full-text">示例 A：该对比视频展示了 Cosmos Transfer 2.5 相较于 Transfer 1 实现的质量提升。视频会在 Transfer 1 结果与 Transfer 2.5 结果之间切换，突出展示更高的视频质量、更好的时间一致性以及更快的推理速度。</span>
        </div>
        <button class="see-more" type="button">展开完整说明</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/av2_t1_t2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">对比</div>
          <span class="preview-text">示例 B：并排对比展示了从 Transfer 1 到 Transfer 2.5 的质量提升。</span>
          <span class="full-text">示例 B：该对比视频展示了 Cosmos Transfer 2.5 相较于 Transfer 1 实现的质量提升。视频会在 Transfer 1 结果与 Transfer 2.5 结果之间切换，突出展示更高的视频质量、更好的时间一致性以及更快的推理速度。</span>
        </div>
        <button class="see-more" type="button">展开完整说明</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/av3_t1_t2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">对比</div>
          <span class="preview-text">示例 C：并排对比展示了从 Transfer 1 到 Transfer 2.5 的质量提升。</span>
          <span class="full-text">示例 C：该对比视频展示了 Cosmos Transfer 2.5 相较于 Transfer 1 实现的质量提升。视频会在 Transfer 1 结果与 Transfer 2.5 结果之间切换，突出展示更高的视频质量、更好的时间一致性以及更快的推理速度。</span>
        </div>
        <button class="see-more" type="button">展开完整说明</button>
      </div>
    </article>

    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="assets/av4_t1_t2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">对比</div>
          <span class="preview-text">示例 D：并排对比展示了从 Transfer 1 到 Transfer 2.5 的质量提升。</span>
          <span class="full-text">示例 D：该对比视频展示了 Cosmos Transfer 2.5 相较于 Transfer 1 实现的质量提升。视频会在 Transfer 1 结果与 Transfer 2.5 结果之间切换，突出展示更高的视频质量、更好的时间一致性以及更快的推理速度。</span>
        </div>
        <button class="see-more" type="button">展开完整说明</button>
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
          // Only set preview if it's empty (preserve values like `{ "seed": 1,`)
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
            toggle.textContent = expanded ? "收起完整说明" : "展开完整说明";
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
        if (originalText.includes("参数")) {
          priorToggle.textContent = "展开完整参数";
        } else {
          priorToggle.textContent = "展开完整说明";
        }
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

---

## 文档信息

**发布日期：**2025 年 11 月 12 日

### 引用

如果您使用了本内容或引用了这项工作，请按以下方式引用：

```bibtex
@misc{cosmos_cookbook_av_gallery_2025,
  title={Autonomous Vehicle Domain Adaptation Gallery},
  author={Wagwani, Raju and Matveiev, Nikolay and Bapst, Joshua and Gu, Jinwei},
  year={2025},
  month={November},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/gallery/av_inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Raju Wagwani、Nikolay Matveiev、Joshua Bapst 和 Jinwei Gu（2025）。《自动驾驶汽车领域自适应示例集》。载于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/gallery/av_inference.html>
