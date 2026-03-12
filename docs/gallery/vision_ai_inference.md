# Vision AI 示例集

> **作者：** [Aiden Chang](https://www.linkedin.com/in/aiden-chang/) • [Akul Santhosh](https://www.linkedin.com/in/akulsanthosh/)

> **机构：** NVIDIA

我们提供了专用的 Brev 实例，帮助您跟随这些示例进行实践。默认配置使用 8× H100 GPU，但您也可以切换为 1× H100 以降低成本（推理速度会更慢）。

[![Brev 实例](./vs_assets/nv-lb-dark.svg)](https://brev.nvidia.com/launchable/deploy/now?launchableID=env-36zmq6sDzikZ1gBSN5Fu3sKezJC)

## 概述

本页展示了使用 Cosmos Transfer 2.5 为 Vision AI 应用生成的结果。这些示例演示了多种城市和道路场景下的 sim-to-real 迁移，说明了如何将源视频转换为体现不同时间段、光照条件、天气、环境效果和场景元素的结果。

如需了解每种控制模态的作用，请参阅我们的[控制模态概念页面](../core_concepts/control_modalities/overview.md)。本页重点展示我们可以生成的一些不同结果。

**使用场景**：基于视觉的应用可以利用这些技术，在无需额外采集数据的情况下，于多样且具有挑战性的条件下训练、测试和验证感知系统。

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

我们展示了该高速公路场景中使用的不同输入控制模态。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_1_short.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">原始 RGB 视频</div>
          <span class="preview-text"></span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_1_edge.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">边缘控制</div>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_1_seg.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">分割控制</div>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_1_depth.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">深度控制</div>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_1_vis.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">Vis 控制</div>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_1_mask.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">使用的掩码</div>
        </div>
      </div>
    </article>
  </div>
</div>

下面展示使用这些控制模态生成的示例结果。

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
      <source src="./vs_assets/vs_fog.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 雾</div>
      <div class="params">guidance: 3, edge: 0.5, depth: 1.0</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a highway with a dense, heavy fog hangs low over the highway, dramatically reducing visibility and softening the outlines of the surrounding hills and leafless trees. A white sedan travels away from the camera in the right lane, its taillights glowing dimly through the fog. The scene conveys slow-moving traffic under conditions with near-whiteout visibility.</div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/vs_morning_sun.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 晨间阳光</div>
      <div class="params">guidance: 3, edge: 1.0, depth: 0.9</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills under clear morning sunlight. The low sun casts long, soft shadows across the gently curving roadway and illuminates dry brown grass and leafless trees along the roadside with a warm, golden glow. A white sedan travels away from the camera in the right lane. The sky is pale blue with thin, high clouds, and the scene captures the calm flow of light traffic in crisp, early-day conditions.</div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/vs_night.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 夜晚</div>
      <div class="params">guidance: 3, edge: 0.5, depth: 1.0</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills at night. The scene is illuminated primarily by vehicle headlights and sparse roadside lighting, with reflective lane markings and road signs glowing against the dark asphalt. The surrounding hills and leafless trees fade into deep shadows beyond the roadway. A white sedan travels away from the camera in the right lane, its red taillights tracing the gentle S-curve. The sky is black and clouded, and the scene conveys light traffic moving steadily through a quiet, nighttime rural environment.</div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/vs_rain.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 雨天</div>
      <div class="params">guidance: 3, edge: 0.9, depth: 1.0</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills, now soaked by a severe rainstorm. The roadway is partially flooded, with standing water pooling across multiple lanes and flowing toward the shoulders, where drainage ditches have overflowed. Dark, rain-slick asphalt reflects headlights and the gray sky above. A white sedan travels away from the camera in the right lane, sending up wide sprays of water. Sheets of rain reduce visibility, and low clouds hang heavy over the scene, conveying hazardous driving conditions during a flood event.
  </div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/vs_snow.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 雪天</div>
      <div class="params">guidance: 3, edge: 0.9, depth: 1.0</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills blanketed in snow, with patches of icy pavement and snowbanks lining the shoulders. Leafless trees are dusted with fresh snow. A white sedan travels away from the camera in the right lane. The scene captures the flow of light traffic under a cold, gray, overcast winter sky.</div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/wooden_road_1.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 木质道路</div>
      <div class="params">guidance: 7, edge: 0.6, seg: 0.4</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills, dry brown grass, and leafless trees. The roadway is constructed from long, weathered wooden planks laid lengthwise, with visible seams, grain patterns, and slight warping between boards. The wooden surface follows the gentle curves of the highway and shows subtle wear from traffic. A white sedan travels away from the camera in the right lane. The scene captures the flow of light traffic on a gray, overcast day.</div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/object.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 障碍物</div>
      <div class="params">guidance: 7, edge: 0.5, seg: 0.8, depth: 0.4</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills, dry brown grass, and leafless trees under a gray, overcast sky. A large brown bear stands in the middle of the roadway near the center divide, facing slightly toward the oncoming lanes.
      </div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/small_car.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 小型汽车</div>
      <div class="params">guidance: 3, edge: 0.5, seg: 0.4, seg_mask: True, depth: 1.0</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills, dry brown grass, and leafless trees. A blue Smart Fortwo microcar travels away from the camera in the right lane, appearing notably small against the wide roadway. The scene captures the flow of light traffic on a gray, overcast day.</div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/van.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 厢式货车</div>
      <div class="params">guidance: 7, edge: 0.5, seg: 0.8, seg_mask: True, depth: 0.5</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills, dry brown grass, and leafless trees. A black Ford Transit cargo van travels away from the camera in the right lane, its tall, boxy profile clearly visible against the wide roadway. The scene captures the flow of light traffic on a gray, overcast day.</div>
    </div>
  </div>
  <div class="masonry-card">
    <video autoplay loop muted playsinline>
      <source src="./vs_assets/people_generation.mp4" type="video/mp4">
      您的浏览器不支持视频标签。
    </video>
    <div class="masonry-overlay">
      <div class="label">参数 - 行人生成</div>
      <div class="params">guidance: 3, edge: 0.5, seg: 0.7, depth: 0.5</div>
      <div class="label">输入提示词</div>
      <div class="prompt">A video of a winding four-lane divided highway cutting through a rural landscape of rolling hills, dry brown grass, and leafless trees. A white sedan travels away from the camera in the right lane. Both sides of the road are lined with wide sidewalks densely populated with pedestrians—dozens of clearly visible people walking in clusters and alone. Individuals wear jackets, hats, and backpacks, some talking to each other, others looking at their phones or walking dogs. The constant movement of people along the sidewalks is a dominant visual element, contrasting with the light vehicle traffic on the road. The scene unfolds under a gray, overcast sky, emphasizing a cool, busy daytime atmosphere.</div>
    </div>
  </div>

</div>

## 仅使用 Edge 与 Depth Control 实现环境变化

该示例演示了如何利用 edge 和 depth control，将视频转换为具有不同环境条件和表面材质的场景。Edge control 保留原始场景结构与运动，depth control 则维持物体之间的空间关系。所有提示词均与上述示例相同。

### 雾天变化

该场景展示了通过调整控制模态生成的不同雾天增强效果。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_fog.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 0.5, depth: 1.0</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_fog_1.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 7, edge: 0.5, depth: 1.0</span>
        </div>
      </div>
    </article>
  </div>
</div>

### 光照变化

该场景展示了通过调整控制模态生成的不同光照条件。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_morning_sun_1.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 0.9, depth: 1.0</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_morning_sun_2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 0.5, depth: 1.0</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_morning_sun.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 1.0, depth: 0.9</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_morning_sun_3.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 7, edge: 0.9, depth: 1.0</span>
        </div>
      </div>
    </article>
  </div>
</div>

### 夜间增强

该场景展示了通过调整控制模态生成的不同夜间条件。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_night.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 0.5, depth: 1.0</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_night_1.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 0.9, depth: 1.0</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_night_2.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 7, edge: 0.5, depth: 1.0</span>
        </div>
      </div>
    </article>
  </div>
</div>

### 雨天增强

该场景展示了通过调整控制模态生成的不同雨天条件。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_rain.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 0.9, depth: 1.0</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_rain_1.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 1.0, depth: 0.9</span>
        </div>
      </div>
    </article>
  </div>
</div>

### 雪天增强

该场景展示了通过调整控制模态生成的不同雪天条件。

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_snow_1.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 1.0</span>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/vs_snow.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 1.0, depth: 0.9</span>
        </div>
      </div>
    </article>
  </div>
</div>

## 其他视频示例

以下是其他相似视频的一些结果。

### 视频示例 1

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_2_short.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">原始 RGB 视频</div>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_2_lighting_augment.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数 - 光照增强</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 1.0, depth: 0.9</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">A video overlooking a roadway intersection in bright morning sunlight. Soft golden light casts long, gentle shadows across the pavement, replacing the earlier overcast atmosphere. In the foreground, a black SUV navigates a sweeping curved lane moving from right to left. Beyond a grassy median, a silver sedan travels along a multi-lane main road that runs past a large concrete building and leafless trees. The scene captures a quiet suburban traffic flow, with crisp visibility and the highway stretching into the distance under a clear early-day sky.</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>
  </div>
</div>

### 视频示例 2

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_3_short.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">原始 RGB 视频</div>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_3_night_augment.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数 - 夜间增强</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 0.5, depth: 1.0</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">A video looking down at a busy multi-lane intersection at night. Streetlights and traffic signals illuminate the scene, casting pools of warm light and reflections on the dark asphalt. Traffic accelerates forward from the stop line, led by a dark gray sedan and a silver sedan, followed closely by a black muscle car with distinctive white racing stripes. To the right, a black SUV turns onto the cross street, passing a red pickup truck parked on the shoulder. In the distance, a large white FedEx truck travels beneath a metal overhead gantry, its headlights and taillights glowing against embankments of dry grass and leafless trees silhouetted in the darkness.</span>
        </div>
        <button class="see-more" type="button">展开完整提示词</button>
      </div>
    </article>
  </div>
</div>

### 视频示例 3

<div class="carousel" data-interval="5000">
  <div class="carousel-track">
    <article class="carousel-slide is-active">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_4_short.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">原始 RGB 视频</div>
        </div>
      </div>
    </article>
    <article class="carousel-slide">
      <div class="media-wrap">
        <video autoplay loop muted playsinline>
          <source src="./vs_assets/clip_4_rain_augment.mp4" type="video/mp4">
          您的浏览器不支持视频标签。
        </video>
        <button class="carousel-btn prev" type="button" aria-label="上一个">‹</button>
        <button class="carousel-btn next" type="button" aria-label="下一个">›</button>
      </div>
      <div class="text-stack">
        <div class="text-block">
          <div class="label">参数 - 雨天增强</div>
          <span class="preview-text"></span>
          <span class="full-text">guidance: 3, edge: 1.0, depth: 0.9</span>
        </div>
        <div class="text-block">
          <div class="label">输入提示词</div>
          <span class="preview-text"></span>
          <span class="full-text">A video overlooking a wide bridge during steady rain. The roadway is darkened and slick with water, reflecting headlights and taillights across multiple lanes of traffic. The weather is gloomy and rainy. </span>
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

    // let timer = slides.length > 1 ? setInterval(next, intervalMs) : null;
    // const resetTimer = () => {
    //   if (!timer) return;
    //   clearInterval(timer);
    //   timer = setInterval(next, intervalMs);
    // };

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

**发布日期：**2025 年 12 月 20 日

### 引用

如果您使用了本内容或引用了这项工作，请按以下方式引用：

```bibtex
@misc{cosmos_cookbook_vision_ai_gallery_2025,
  title={Vision AI Gallery},
  author={Chang, Aiden and Santhosh, Akul},
  year={2025},
  month={December},
  howpublished={\url{https://nvidia-cosmos.github.io/cosmos-cookbook/gallery/vision_ai_inference.html}},
  note={NVIDIA Cosmos Cookbook}
}
```

**建议的文本引用：**

> Aiden Chang 和 Akul Santhosh（2025）。《Vision AI 示例集》。载于 *NVIDIA Cosmos Cookbook*。访问地址：<https://nvidia-cosmos.github.io/cosmos-cookbook/gallery/vision_ai_inference.html>
