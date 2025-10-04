---
layout: fullscreen
title: Chromatic Mindscapes
tags:
  - graphics
---

<canvas id="psyCanvas" style="width:100vw; height:100vh; display:block; position:fixed; top:0; left:0;"></canvas>
<script>
const canvas = document.getElementById('psyCanvas');
const ctx = canvas.getContext('2d');

function resize() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
resize();
window.addEventListener('resize', resize);

// Utility for smooth variation
function ease(t) {
  return 0.5 - 0.5 * Math.cos(Math.PI * t);
}

function hsvToRgb(h, s, v) {
  let f = (n, k = (n + h / 60) % 6) =>
    v - v * s * Math.max(Math.min(k, 4 - k, 1), 0);
  return [f(5), f(3), f(1)];
}

let time = 0;

function drawPsychedelic() {
  const w = canvas.width;
  const h = canvas.height;
  const image = ctx.getImageData(0, 0, w, h);
  const data = image.data;

  time += 0.012;

  // Parameters that evolve over time
  const cx = w / 2 + Math.sin(time * 0.7) * (w / 5);
  const cy = h / 2 + Math.cos(time * 0.9) * (h / 5);
  const freq1 = 0.006 + Math.sin(time * 0.82) * 0.004;
  const freq2 = 0.009 + Math.cos(time * 1.03) * 0.002;
  const phase = time * 2;
  const scale = 0.8 + 0.2 * Math.sin(time * 0.63);

  for (let y = 0; y < h; ++y) {
    for (let x = 0; x < w; ++x) {
      const dx = (x - cx) * scale;
      const dy = (y - cy) * scale;
      const r = Math.sqrt(dx * dx + dy * dy);
      // Multi-layered, time-evolving field
      let angle = Math.atan2(dy, dx);
      let v1 = Math.sin(r * freq1 + angle * 2.2 + phase);
      let v2 = Math.cos(r * freq2 - angle * 3.1 - phase * 0.7);
      let v3 = Math.sin((dx + dy) * 0.012 + time * 1.1);
      let v = ease((v1 + v2 + v3 + 3) / 6);

      let hue = ((v1 * 0.5 + 0.5) * 360 + time * 40 + r * 0.07) % 360;
      let sat = 0.85 + 0.15 * v2;
      let val = 0.52 + 0.48 * v;

      let [rr,gg,bb] = hsvToRgb(hue, sat, val);

      let px = 4 * (y * w + x)
      data[px    ] = Math.floor(rr * 255);
      data[px + 1] = Math.floor(gg * 255);
      data[px + 2] = Math.floor(bb * 255);
      data[px + 3] = 255;
    }
  }

  ctx.putImageData(image,0,0);
}

function animate() {
  drawPsychedelic();
  requestAnimationFrame(animate);
}
animate();
</script>
