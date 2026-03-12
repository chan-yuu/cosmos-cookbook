# 全部配方

<style>
  .recipe-page {
    --recipe-right-bleed: clamp(2rem, 12vw, 18rem);
    width: calc(100% + var(--recipe-right-bleed));
    margin-right: calc(-1 * var(--recipe-right-bleed));
    padding-right: 0.5rem;
  }

  .recipe-board {
    display: flex;
    flex-direction: column;
    gap: 2rem;
    margin-top: 1.5rem;
  }

  .recipe-intro {
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
    margin-bottom: 1.5rem;
  }

  .recipe-intro p {
    margin: 0;
  }

  .recipe-category {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }

  .recipe-category-body {
    padding: 1.1rem;
    border: 1px solid rgba(255, 255, 255, 0.35);
    background: var(--md-default-bg-color, #111111);
    box-shadow: 0 8px 28px rgba(0, 0, 0, 0.25);
    border-radius: 0;
  }

  .category-header {
    display: flex;
    flex-direction: column;
    gap: 0.35rem;
    /*Stretch header text to full available width so it wraps later (matches carousel width).*/
    align-items: stretch;
  }

  .category-header h2 {
    margin: 0;
  }

  .category-header p {
    margin: 0;
    color: var(--md-default-fg-color--light, #b7bec8);
    /*Don't artificially constrain the description width; let it use the full content width.*/
    max-width: none;
  }

  .recipe-track {
    display: flex;
    gap: 1rem;
    overflow-x: auto;
    padding: 0.25rem 0.25rem 0.75rem;
    scroll-snap-type: x mandatory;
    align-items: stretch;
    position: relative;
    z-index: 0;
  }

  .recipe-track::-webkit-scrollbar {
    height: 8px;
  }

  .recipe-track::-webkit-scrollbar-track {
    background: rgba(255, 255, 255, 0.08);
  }

  .recipe-track::-webkit-scrollbar-thumb {
    background: var(--md-accent-fg-color, #76b900);
    border-radius: 0;
  }

  .recipe-card {
    flex: 0 0 250px;
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
    padding: 0.8rem;
    border: 1px solid rgba(255, 255, 255, 0.35);
    text-decoration: none;
    color: var(--md-default-fg-color, #f2f2f2);
    background: var(--md-default-bg-color, #111111);
    scroll-snap-align: start;
    transition: border-color 150ms ease, transform 150ms ease, box-shadow 150ms ease;
    border-radius: 0;
  }

  .recipe-card:hover {
    border-color: var(--md-accent-fg-color, #76b900);
    transform: translateY(-2px);
    box-shadow: 0 10px 24px rgba(0, 0, 0, 0.3);
  }

  .recipe-media {
    width: 100%;
    height: 150px;
    border-radius: 0;
    border: 1px solid rgba(255, 255, 255, 0.18);
    background: repeating-linear-gradient(
      135deg,
      rgba(118, 185, 0, 0.1),
      rgba(118, 185, 0, 0.1) 14px,
      rgba(118, 185, 0, 0.2) 14px,
      rgba(118, 185, 0, 0.2) 28px
    );
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--md-default-fg-color--light, #b7bec8);
    font-weight: 700;
    letter-spacing: 0.02em;
    position: relative;
    overflow: hidden;
  }

  /*For cards where we show a real image, use a clean background instead of the placeholder pattern.*/
  .recipe-media--image {
    background: #ffffff;
  }

  .recipe-media img {
    width: 100%;
    height: 100%;
    /*Avoid cropping hero images in carousels.*/
    object-fit: contain;
    /*When `object-fit: contain` letterboxes, keep the empty area white in dark mode too.*/
    background: #ffffff;
    display: block;
  }

  .recipe-media--video {
    /*Match image cards: white background so letterboxing isn't black.*/
    background: #ffffff;
  }

  .recipe-media video {
    width: 100%;
    height: 100%;
    object-fit: contain;
    /*Match image cards: white background so letterboxing isn't black.*/
    background: #ffffff;
    display: block;
    /*Ensure the whole card remains clickable; video shouldn't capture pointer events.*/
    pointer-events: none;
  }

  .recipe-title {
    font-weight: 700;
    line-height: 1.3;
    color: var(--md-accent-fg-color, #76b900);
  }

  .recipe-tag {
    align-self: flex-start;
    margin-top: auto;
    display: inline-flex;
    align-items: center;
    padding: 0.15rem 0.4rem;
    font-size: 0.6rem;
    font-weight: 600;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    line-height: 1.1;
    border-radius: 0;
    color: #ffffff;
  }

  .recipe-tag--inference {
    background-color: #76b900;
  }

  .recipe-tag--post-training {
    background-color: #4aa3ff;
  }

  .recipe-tag--curation {
    background-color: #f5a623;
  }

  .recipe-tag--workflow {
    background-color: #36c2b2;
  }

  .md-sidebar--secondary {
    display: none;
  }

  .md-content__inner {
    overflow: visible;
  }

  @media (max-width: 640px) {
    .recipe-card {
      flex-basis: 210px;
    }
  }
</style>

<div class="recipe-page">
  <div class="recipe-intro">
    <p>在一个页面中探索覆盖各个领域的 Cosmos 配方。</p>
  </div>

  <div class="recipe-board">
  <section class="recipe-category" id="robotics">
    <div class="category-header">
      <h2>机器人</h2>
      <p>面向机器人训练的操作、导航和具身推理工作流。</p>
    </div>
    <div class="recipe-category-body">
      <div class="recipe-track" data-page-size="6" aria-label="机器人配方">
        <a class="recipe-card" href="./end2end/gr00t-dreams/post-training.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video autoplay loop muted playsinline preload="none" tabindex="-1">
              <source src="./end2end/gr00t-dreams/assets/3.mp4" type="video/mp4">
              您的浏览器不支持 video 标签。
            </video>
          </div>
          <div class="recipe-title">GR00T-Dreams：用于机器人学习的合成轨迹生成</div>
          <div class="recipe-tag recipe-tag--workflow">工作流</div>
        </a>
        <a class="recipe-card" href="./post_training/predict2/cosmos_policy/post_training.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video autoplay loop muted playsinline preload="none" tabindex="-1">
              <source src="./post_training/predict2/cosmos_policy/assets/aloha_rollouts/fold_shirt_15x_speed.mp4" type="video/mp4">
              您的浏览器不支持 video 标签。
            </video>
          </div>
          <div class="recipe-title">Cosmos Policy：面向视觉运动控制与规划的视频模型微调</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./inference/reason2/intbot_showcase/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/reason2/intbot_showcase/assets/IntBot-GTC.jpg" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos-Reason2-8B 进行第一人称社交与物理推理</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer1/inference-warehouse-mv/inference.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video src="./inference/transfer1/inference-warehouse-mv/assets/combined_grid_rgb.mp4" autoplay muted loop loading="lazy"></video>
          </div>
          <div class="recipe-title">Cosmos Transfer 1 用于多视角仓库检测与跟踪的 Sim2Real</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer1/gr00t-mimic/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/transfer1/gr00t-mimic/assets/hero.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">Isaac GR00T-Mimic 用于合成操作动作生成</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer1/inference-x-mobility/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/transfer1/inference-x-mobility/assets/output_xmob.gif" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">Cosmos Transfer 用于机器人导航任务的 Sim2Real</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./post_training/predict2/gr00t-dreams/post-training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/predict2/gr00t-dreams/assets/hero.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">Isaac GR00T-Dreams 用于合成轨迹数据生成</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason1/spatial-ai-warehouse/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason1/spatial-ai-warehouse/assets/data_overview.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 1 进行面向仓库场景的 Spatial AI 后训练</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason1/temporal_localization/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason1/temporal_localization/assets/cube_stacking.gif" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 进行 Mimic Gen 时序定位</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
      </div>
    </div>
  </section>

  <section class="recipe-category" id="autonomous-vehicles">
    <div class="category-header">
      <h2>自动驾驶汽车</h2>
      <p>仿真、交通场景，以及面向自动驾驶规模的数据生成与评估。</p>
    </div>
    <div class="recipe-category-body">
      <div class="recipe-track" data-page-size="6" aria-label="自动驾驶汽车配方">
        <a class="recipe-card" href="./post_training/reason2/av_3d_grounding/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason2/av_3d_grounding/assets/training_images_overlay_overview.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 1 和 2 进行 3D AV Grounding 后训练</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason2/video_caption_vqa/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason2/video_caption_vqa/assets/mcq_vqa_results.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">为 AV 视频描述与 VQA 后训练 Cosmos Reason 2</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/transfer2_5/av_world_scenario_maps/post_training.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video autoplay loop muted playsinline preload="none" tabindex="-1">
              <source src="./post_training/transfer2_5/av_world_scenario_maps/assets/av_rgb_front_wide.mp4" type="video/mp4">
              您的浏览器不支持 video 标签。
            </video>
          </div>
          <div class="recipe-title">使用 World Scenario Map 控制的 Cosmos Transfer 2.5 多视角生成</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./inference/predict2/inference-its/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/predict2/inference-its/assets/output.jpg" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">Cosmos Predict 2 用于智能交通系统 (ITS) 图像的 Text2Image</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer1/inference-its-weather-augmentation/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/transfer1/inference-its-weather-augmentation/assets/rainy_night_all_09.jpg" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">Cosmos Transfer 1 用于智能交通系统 (ITS) 图像的天气增强</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer2_5/inference-carla-sdg-augmentation/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/transfer2_5/inference-carla-sdg-augmentation/assets/augmentation_matrix_grid.gif" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">Cosmos Transfer 2.5 用于仿真器视频的 Sim2Real</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./post_training/predict2/its-accident/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/transfer2_5/inference-carla-sdg-augmentation/assets/augmented_anomaly_trajectory.gif" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Predict2 生成交通异常</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason2/intelligent-transportation/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason2/intelligent-transportation/assets/after_qa.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 2 进行智能交通后训练</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason1/intelligent-transportation/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason1/intelligent-transportation/assets/e2e_workflow.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 1 进行智能交通后训练</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason1/av_video_caption_vqa/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason1/av_video_caption_vqa/assets/sft_results.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">用于 AV 视频描述与 VQA 的 SFT</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./end2end/smart_city_sdg/workflow_e2e.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./end2end/smart_city_sdg/assets/main_workflow.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">面向交通场景的合成数据生成 (SDG)</div>
          <div class="recipe-tag recipe-tag--workflow">工作流</div>
        </a>
      </div>
    </div>
  </section>

  <section class="recipe-category" id="vision-ai">
    <div class="category-header">
      <h2>视觉 AI</h2>
      <p>覆盖图像与视频模态的视觉生成、数据整理和领域迁移。</p>
    </div>
    <div class="recipe-category-body">
      <div class="recipe-track" data-page-size="6" aria-label="视觉 AI 配方">
        <a class="recipe-card" href="./inference/reason2/worker_safety/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/reason2/worker_safety/assets/assets_1_worker_safety.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 2 的传统仓库工人安全分析</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/reason2/vss/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/reason2/vss/assets/warehouse_summary_example.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 进行视频搜索与摘要</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./data_curation/embedding_analysis/embedding_analysis.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./data_curation/embedding_analysis/assets/clusters.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">基于嵌入的 Time Series K-Means 数据集视频聚类</div>
          <div class="recipe-tag recipe-tag--curation">数据整理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer2_5/biotrove_augmentation/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/transfer2_5/biotrove_augmentation/assets/moth_biotrove.webp" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Transfer 2.5 为 BioTrove 飞蛾进行领域迁移</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer2_5/inference-real-augmentation/inference.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./inference/transfer2_5/inference-real-augmentation/assets/omniverse_background_change_recipe.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Transfer 2.5 的多控制配方</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./inference/transfer2_5/inference-image-prompt/inference.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video autoplay loop muted playsinline preload="none" tabindex="-1">
              <source src="./inference/transfer2_5/inference-image-prompt/assets/example1_generation-from-edge-sunset.mp4" type="video/mp4">
              您的浏览器不支持 video 标签。
            </video>
          </div>
          <div class="recipe-title">使用 Cosmos Transfer 2.5 的风格引导视频生成</div>
          <div class="recipe-tag recipe-tag--inference">推理</div>
        </a>
        <a class="recipe-card" href="./post_training/predict2_5/sports/post_training.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video autoplay loop muted playsinline preload="none" tabindex="-1">
              <source src="./post_training/predict2_5/sports/assets/post_trained/12.mp4" type="video/mp4">
              您的浏览器不支持 video 标签。
            </video>
          </div>
          <div class="recipe-title">面向体育视频生成的 LoRA 后训练</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason2/physical-plausibility-check/post_training.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video autoplay loop muted playsinline preload="none" tabindex="-1">
              <source src="https://videophysics2trainvideos.s3.us-east-2.amazonaws.com/hunyuan_xedit_train/A_robotic_arm_gently_pokes_a_stack_of_plastic_cups,_making_the_bottom_cups_slide_out_and_the_whole_stack_fall.mp4" type="video/mp4">
              您的浏览器不支持 video 标签。
            </video>
          </div>
          <div class="recipe-title">使用 Cosmos Reason 2 进行物理合理性预测</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason1/physical-plausibility-check/post_training.html">
          <div class="recipe-media recipe-media--video" aria-hidden="true">
            <video autoplay loop muted playsinline preload="none" tabindex="-1">
              <source src="https://videophysics2testvideos.s3.us-east-2.amazonaws.com/hunyuan_xdit/A_car_crashes_into_a_stack_of_cardboard_boxes,_sending_the_boxes_flying_in_all_directions.mp4" type="video/mp4">
              您的浏览器不支持 video 标签。
            </video>
          </div>
          <div class="recipe-title">使用 Cosmos Reason 1 进行物理合理性预测</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./post_training/reason1/wafermap_classification/post_training.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="./post_training/reason1/wafermap_classification/assets/Picture8.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Reason 1 进行晶圆图异常分类</div>
          <div class="recipe-tag recipe-tag--post-training">后训练</div>
        </a>
        <a class="recipe-card" href="./data_curation/predict2_data/data_curation.html">
          <div class="recipe-media recipe-media--image" aria-hidden="true">
            <img src="../core_concepts/data_curation/images/grid_preview.png" alt="" loading="lazy" />
          </div>
          <div class="recipe-title">使用 Cosmos Curator 为 Cosmos Predict 微调整理数据</div>
          <div class="recipe-tag recipe-tag--curation">数据整理</div>
        </a>
      </div>
    </div>
  </section>
</div>
</div>
