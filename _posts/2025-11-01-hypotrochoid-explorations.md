---
layout: post
title: "Hypotrochoid Explorations"
tags:
  - hypotrochoid
  - spirograph
  - geometry
  - interactive
date: 2025-11-01
---

A hypotrochoid is the curve traced by a point attached to a circle rolling inside a larger fixed circle. It's the math behind every Spirograph kit.

## Interactive Demo

<style>
@media print {
  body * { visibility: hidden; }
  #canvas-wrap, #canvas-wrap * { visibility: visible; }
  #canvas-wrap {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    background: #fff;
  }
  #canvas-wrap canvas { max-width: 100vw; max-height: 100vh; width: auto; height: auto; }
}

#canvas-wrap {
  position: relative;
  display: inline-block;
  cursor: pointer;
  border: 1px solid #333;
  border-radius: 8px;
  overflow: hidden;
  line-height: 0;
}

#canvas-wrap.fullscreen {
  position: fixed;
  top: 0; left: 0; width: 100vw; height: 100vh;
  background: #000;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 0;
}

#canvas-wrap .close-btn {
  display: none;
}
#canvas-wrap.fullscreen .close-btn {
  display: block;
  position: absolute;
  top: 12px; right: 16px;
  color: #fff;
  font-size: 28px;
  line-height: 1;
  cursor: pointer;
  z-index: 1;
  user-select: none;
}
</style>

<div id="canvas-wrap">
  <span class="close-btn">&times;</span>
  <canvas id="canvas"></canvas>
</div>

<script>
const wrap = document.getElementById('canvas-wrap');
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

let W, H, cx, cy, R, r, d, t = 0;

function size() {
  const full = wrap.classList.contains('fullscreen');
  const dpr = window.devicePixelRatio || 1;
  W = full ? window.innerWidth : 600;
  H = full ? window.innerHeight : 600;
  canvas.width = W * dpr;
  canvas.height = H * dpr;
  canvas.style.width = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  cx = W / 2; cy = H / 2;
  R = Math.min(W, H) * 0.30;
  r = R * 0.39;
  d = R * 0.67;
}
size();

function draw() {
  ctx.clearRect(0, 0, W, H);
  ctx.beginPath();
  for (let i = 0; i < 2000; i++) {
    const a = (i / 2000) * Math.PI * 20 + t;
    const x = cx + (R - r) * Math.cos(a) + d * Math.cos(((R - r) / r) * a);
    const y = cy + (R - r) * Math.sin(a) - d * Math.sin(((R - r) / r) * a);
    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.strokeStyle = `hsl(${(t * 30) % 360}, 80%, 60%)`;
  ctx.lineWidth = 1.5;
  ctx.stroke();
  t += 0.005;
  requestAnimationFrame(draw);
}
draw();

let lastTap = 0;
wrap.addEventListener('pointerdown', function(e) {
  const now = Date.now();
  if (now - lastTap < 400) {
    e.preventDefault();
    wrap.classList.toggle('fullscreen');
    size();
  }
  lastTap = now;
});

wrap.addEventListener('dblclick', function(e) {
  wrap.classList.toggle('fullscreen');
  size();
});

document.addEventListener('keydown', function(e) {
  if (e.key === 'Escape' && wrap.classList.contains('fullscreen')) {
    wrap.classList.remove('fullscreen');
    size();
  }
});

wrap.querySelector('.close-btn').addEventListener('click', function(e) {
  e.stopPropagation();
  wrap.classList.remove('fullscreen');
  size();
});
</script>

## The Math

Given fixed circle radius **R**, rolling circle radius **r**, and pen distance **d** from the rolling center:

- **x** = (R − r) cos(θ) + d cos(((R − r) / r) θ)
- **y** = (R − r) sin(θ) − d sin(((R − r) / r) θ)

Varying the ratios produces an endless family of hypnotic patterns.
