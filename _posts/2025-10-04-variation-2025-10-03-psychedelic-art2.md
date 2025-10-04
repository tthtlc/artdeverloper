```markdown
---
layout: fullscreen
title: "Fractal Wave Spirals"
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

// Recursive function to draw a wavy fractal spiral arm
function drawSpiralArm(cx, cy, angle, length, level, time, baseHue) {
  if (level > 5 || length < 4) return;
  // Wavy spiral offset for this recursion
  const wave = Math.sin(time * 0.012 + length * 0.09 + level * 12) * 14 * (6 - level);

  // The position at the end of the current segment
  const nx = cx + Math.cos(angle) * length + Math.cos(time * 0.005 + level * 3) * 3;
  const ny = cy + Math.sin(angle) * length + Math.sin(time * 0.005 + level * 3) * 3;

  // Colorful gradient along the spiral
  let hue = (baseHue + angle * 120 / Math.PI + level * 22 + Math.sin(time*0.01+level)*60) % 360;
  ctx.strokeStyle = `hsl(${hue}, 95%, 53%)`;
  ctx.lineWidth = Math.max(1.5, 9 - level * 2);

  ctx.beginPath();
  ctx.moveTo(cx, cy);
  ctx.lineTo(nx, ny);
  ctx.stroke();

  // Pulsing dots along the spiral arm
  for (let j = 0; j < 3; j++) {
    let dotAngle = angle + wave * 0.001 * j;
    let dotR = length * (j+1)/3;
    let dx = cx + Math.cos(dotAngle) * dotR;
    let dy = cy + Math.sin(dotAngle) * dotR;
    let dSize = 5 + Math.sin(time*0.03 + dotR + j + level) * 4 * (1 + 0.2*level);

    let dhue = (hue + 60*j + Math.sin(time*0.03+j*7+level)*30) % 360;
    ctx.beginPath();
    ctx.arc(dx, dy, dSize, 0, 2 * Math.PI);
    ctx.fillStyle = `hsla(${dhue}, 100%, 66%,0.68)`;
    ctx.shadowColor = `hsl(${dhue},90%,70%)`;
    ctx.shadowBlur = 18 - level * 2;
    ctx.fill();
    ctx.shadowBlur = 0;
  }

  // Branch into two new spiral arms with slightly different angles and reduced length
  drawSpiralArm(
    nx, ny,
    angle + 0.32 + Math.sin(time * 0.004 + level) * 0.2,
    length * 0.78 + Math.sin(time * 0.025 + level * 3) * 3,
    level + 1,
    time,
    baseHue + 27
  );
  drawSpiralArm(
    nx, ny,
    angle - 0.29 + Math.sin(time * 0.004 + level + 10)*0.17,
    length * 0.81 + Math.cos(time * 0.023 + level * 4) * 2,
    level + 1,
    time,
    baseHue - 19
  );
}

function draw() {
  const w = canvas.width, h = canvas.height;
  // Soft fade for trailing afterimages
  ctx.globalAlpha = 0.19 + 0.08 * Math.sin(t*0.021);
  ctx.fillStyle = "rgba(0,0,0,1)";
  ctx.fillRect(0,0,w,h);
  ctx.globalAlpha = 1;

  const cx = w/2, cy = h/2;
  const spiralCount = 7;
  const baseRad = Math.min(w, h) * 0.24;
  let rot = t * 0.007 + Math.sin(t*0.023)*0.06;

  // Draw multiple rotating spiral fractals, each shifted by offset
  for(let i=0; i<spiralCount; i++) {
    let theta = rot + i * (Math.PI * 2 / spiralCount) + Math.cos(t*0.014+i*14)*0.4;
    // Pulsing radii for each spiral
    let sRad = baseRad + 60 * Math.sin(t*0.028 + i * 2.9);

    drawSpiralArm(
      cx + Math.cos(theta) * sRad * 0.18,
      cy + Math.sin(theta) * sRad * 0.18,
      theta + Math.cos(t*0.03+i*8)*0.5,
      sRad,
      0,
      t,
      (i * 360 / spiralCount + t*0.7) % 360
    );
  }
  t++;
  requestAnimationFrame(draw);
}

draw();
</script>
```
