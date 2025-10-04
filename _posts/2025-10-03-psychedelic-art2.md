---
layout: post
title: "Psychedelic Art Generator"
tags:
  - graphics, graphics, digital arts, mathematical art, Javascript
---

<style>
/* Hide Jekyll layout elements for fullscreen effect */
.navbar, .intro-header, .post-container, .sidebar-container, footer {
  display: none !important;
}

<style>
    body {
      margin: 0;
      overflow: hidden;
      background: black;
    }
    canvas {
      display: block;
    }
</style>
<canvas id="canvas"></canvas>
<script>
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

function resizeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
resizeCanvas();
window.addEventListener("resize", resizeCanvas);

let t = 0;

function draw() {
  const w = canvas.width;
  const h = canvas.height;

  // Fade previous frame slightly to create trails
  ctx.fillStyle = "rgba(0, 0, 0, 0.1)";
  ctx.fillRect(0, 0, w, h);

  const cx = w / 2;
  const cy = h / 2;

  const numShapes = 200;
  for (let i = 0; i < numShapes; i++) {
    const angle = (i / numShapes) * Math.PI * 2 + t * 0.01;
    const radius = Math.sin(t * 0.02 + i) * 250 + 300;

    const x = cx + Math.cos(angle) * radius;
    const y = cy + Math.sin(angle) * radius;

    const hue = (i * 12 + t * 4) % 360;
    const size = 10 + 8 * Math.sin(t * 0.05 + i);

    ctx.beginPath();
    ctx.arc(x, y, size, 0, Math.PI * 2);
    ctx.fillStyle = `hsl(${hue}, 100%, 50%)`;
    ctx.fill();
  }

  t++;
  requestAnimationFrame(draw);
}

draw();
</script>
