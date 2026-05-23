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

<div>
<canvas id="canvas" width="600" height="600" style="border:1px solid #333; border-radius: 8px;"></canvas>
</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const cx = 300, cy = 300;
const R = 180, r = 70, d = 120;
let t = 0;

function draw() {
  ctx.clearRect(0, 0, 600, 600);
  ctx.beginPath();
  for (let i = 0; i < 2000; i++) {
    const angle = (i / 2000) * Math.PI * 20 + t;
    const x = cx + (R - r) * Math.cos(angle) + d * Math.cos(((R - r) / r) * angle);
    const y = cy + (R - r) * Math.sin(angle) - d * Math.sin(((R - r) / r) * angle);
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
</script>

## The Math

Given fixed circle radius **R**, rolling circle radius **r**, and pen distance **d** from the rolling center:

- **x** = (R − r) cos(θ) + d cos(((R − r) / r) θ)
- **y** = (R − r) sin(θ) − d sin(((R − r) / r) θ)

Varying the ratios produces an endless family of hypnotic patterns.
