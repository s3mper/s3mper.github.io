---
layout: post
comments: true

title: "Umbrella Text Reflow: Pretext 텍스트 배치 데모"
excerpt: "Pretext Library Demo Site"

date: 2026-04-16
last_modified_at: 2026-04-16

categories: [javascript]
tags: [demo, pretext, ㅓㄴ]
redirect_from:
  - /Umbrella-Reflow/
guid: https://s3mper.github.io/Umbrella-Reflow/
---

<!-- markdownlint-disable MD025 MD033 -->

<link rel="stylesheet" href="{{ '/assets/css/umbrella-reflow.css' | relative_url }}">

Move your cursor to drag the umbrella around. Text falls like rain in vertical columns — characters under the umbrella are blocked, creating the sheltered zone. On mobile, the umbrella sways automatically.

<div class="wind-control">
  <label for="wind-strength">Wind</label>
  <input
    id="wind-strength"
    type="range"
    min="-1"
    max="1"
    step="0.01"
    value="0"
  >
  <span id="wind-strength-value">0.00</span>
</div>

<section class="umbrella-reflow-section">
  <canvas
  id="umbrella-rain-canvas"
  data-umbrella-src="{{ '/assets/img/2026-04-16-umbrella.svg' | relative_url }}"
></canvas>
</section>

<script type="module" src="{{ '/assets/js/umbrella-reflow.js' | relative_url }}"></script>

pretext demo
