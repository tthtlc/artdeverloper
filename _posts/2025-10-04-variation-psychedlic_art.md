---
layout: fullscreen
title: "Psychedelic Hexagonal Wave Vortex"
tags:
  - graphics
---

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

// Helper to draw a hexagon at (x, y) with given size
function hexagon(x, y, size) {
  ctx.beginPath();
  for (let i = 0; i < 6; i++) {
    const angle = Math.PI / 3 * i;
    const px = x + Math.cos(angle) * size;
    const py = y + Math.sin(angle) * size;
    if (i === 0) ctx.moveTo(px, py);
    else ctx.lineTo(px, py);
  }
  ctx.closePath();
}

function draw() {
  const w = canvas.width;
  const h = canvas.height;
  const cx = w / 2;
  const cy = h / 2;

  // Fade previous frames for trails & blending
  ctx.globalCompositeOperation = "lighter";
  ctx.fillStyle = "rgba(0, 0, 0, 0.08)";
  ctx.fillRect(0, 0, w, h);
  ctx.globalCompositeOperation = "source-over";

  // Hex grid parameters
  const baseSize = Math.min(w, h) * 0.034;
  const hexHeight = Math.sqrt(3) * baseSize;
  const cols = Math.floor(w / (baseSize * 1.5)) + 3;
  const rows = Math.floor(h / hexHeight) + 3;

  for (let row = -1; row < rows; row++) {
    for (let col = -1; col < cols; col++) {
      // Hex grid layout
      let x = col * baseSize * 1.5;
      let y = row * hexHeight + (col % 2) * (hexHeight / 2);

      // Center to canvas
      x = x - w / 2;
      y = y - h / 2;

      // Polar coordinates for vortex motion
      const dist = Math.sqrt(x * x + y * y);
      const a = Math.atan2(y, x);

      // Psychedelic transformation: swirl, pulse, and wave
      const angleOffset = Math.cos(dist * 0.012 + t * 0.008) * 1.6 + Math.sin(dist * 0.015 - t * 0.005) * 1.3;
      const spiralRadius = dist * (0.95 + 0.17 * Math.sin(t * 0.007 + dist * 0.015));
      const swirlA = a + angleOffset + Math.sin(t * 0.01 + dist * 0.007);

      // Dynamic position
      const px = cx + Math.cos(swirlA) * (spiralRadius);
      const py = cy + Math.sin(swirlA) * (spiralRadius);

      // Dynamic hexagon size and rotation
      const pulse = 0.4 + 0.55 * Math.sin(t * 0.016 + dist * 0.021 + Math.cos(a * 2.6));
      const size = baseSize * pulse;

      // Vibrating color waves based on angles, time, and grid
      const hue = (a * 120 + dist * 0.5 + t * 2.5 + 310 + 110 * Math.cos(row + col + t * 0.017)) % 360;
      const lightness = 43 + 26 * Math.sin(dist * 0.018 - t * 0.009);

      // Draw hexagon
      ctx.save();
      ctx.translate(px, py);
      ctx.rotate(swirlA + t * 0.01 + (row + col) * 0.04);
      hexagon(0, 0, size);
      ctx.strokeStyle = `hsla(${hue}, 90%, ${lightness}%, 0.85)`;
      ctx.lineWidth = 1.2 + 0.8 * Math.sin(t * 0.011 + col - row);
      ctx.shadowColor = `hsl(${(hue+100)%360}, 95%, 60%)`;
      ctx.shadowBlur = 9 + 8 * Math.cos(a + t * 0.02);
      ctx.globalAlpha = 0.85;
      ctx.stroke();
      ctx.globalAlpha = 1;
      ctx.shadowBlur = 0;
      ctx.restore();
    }
  }

  t += 1;
  requestAnimationFrame(draw);
}
draw();
</script>
