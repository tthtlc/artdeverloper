---
layout: fullscreen
title: "Psychedelic Radiant Waves: Nested Pulsating Flower of Life"
tags:
  - graphics
---

<canvas id="psyCanvas" width="800" height="800" style="background:#11031b;display:block;margin:2em auto;border-radius:12px;"></canvas>
<script>
const canvas = document.getElementById('psyCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;
const CX = W/2, CY = H/2;

const PETALS = 12;  // number of petals / lobes
const LAYERS = 12;  // nested shapes
const POINTS = 240; // smoothness of curves

let t = 0;

// Generate a single layered petal shape (radial "lobe" polygon)
function genLayerPoints(layer, time) {
  const R_base = 110 + 50 * Math.sin(time/1.7 + layer*0.37);
  const amp = 44 + 18*Math.sin(time/2.4 + layer*1.3);
  const freq = PETALS + layer*0.5 + 1.5*Math.sin(layer*2 + time/3.1);
  const twist = (layer/3 + 1.7) * Math.sin(time/2 + layer);

  let arr = [];
  for (let i=0; i<POINTS; i++) {
    const theta = (i/POINTS) * Math.PI*2;
    const pulse = Math.sin(freq*theta + twist + time/2.3 + layer);
    let R = R_base + amp * pulse;
    // Add subtle oscillation per point for jittery psychedelic effect
    R += 14 * Math.sin(time + theta*3 + layer*0.8);
    let x = CX + R * Math.cos(theta);
    let y = CY + R * Math.sin(theta);
    arr.push({x, y});
  }
  return arr;
}

function drawLayer(layer, time) {
  // `layer` goes from 0 (innermost) to LAYERS-1 (outermost)
  const shape = genLayerPoints(layer, time);
  ctx.save();
  // Color: rainbow, with layer depth and motion
  const hue = (360*layer/LAYERS + 120 * Math.sin(time/2.6 + layer/2) + 200)%360;
  const sat = 85 + 14 * Math.sin(layer + time/6.5);
  const lum = 37 + 22 * Math.sin(time/2.1 + layer/3);

  // Psychedelic blend mode overlays
  ctx.globalCompositeOperation = "lighter";
  ctx.strokeStyle = `hsl(${hue},${sat}%,${lum+14}%)`;
  ctx.lineWidth = 2.3 + 2*Math.sin(time/3 + layer*0.6);
  ctx.beginPath();
  ctx.moveTo(shape[0].x, shape[0].y);
  for (let i=1; i<shape.length; i++) {
    ctx.lineTo(shape[i].x, shape[i].y);
  }
  ctx.closePath();
  ctx.shadowColor = `hsl(${hue},88%,83%)`;
  ctx.shadowBlur = 40 + 5*Math.sin(time-layer*0.47);
  ctx.stroke();
  ctx.restore();

  // Glow fill
  ctx.save();
  ctx.globalCompositeOperation = "lighter";
  ctx.globalAlpha = 0.035 + 0.012 * Math.sin(time/4 + layer);
  ctx.fillStyle = `hsl(${(hue+60)%360},100%,96%)`;
  ctx.beginPath();
  ctx.moveTo(shape[0].x, shape[0].y);
  for (let i=1; i<shape.length; i++) ctx.lineTo(shape[i].x, shape[i].y);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}

function animate() {
  ctx.clearRect(0, 0, W, H);
  // Dim previous frame for ghosting trails
  ctx.save();
  ctx.globalAlpha = 0.22;
  ctx.globalCompositeOperation = "source-over";
  ctx.fillStyle = "#11031b";
  ctx.fillRect(0,0,W,H);
  ctx.restore();

  for(let l=LAYERS-1; l>=0; l--){
    drawLayer(l, t-l*0.33);
  }

  // Center dynamic sun-dot
  ctx.save();
  const sunRad = 16 + 10 * Math.abs(Math.sin(t/2.1));
  ctx.globalAlpha = 0.43 + 0.19*Math.sin(t);
  ctx.shadowColor = "#fff9df";
  ctx.shadowBlur = 70;
  ctx.beginPath();
  ctx.arc(CX, CY, sunRad, 0, Math.PI*2);
  ctx.fillStyle = `hsl(${(150+57*Math.sin(t*2))%360},90%,88%)`;
  ctx.fill();
  ctx.restore();

  t += 0.018;
  requestAnimationFrame(animate);
}

// Responsive resizing for aesthetics
window.addEventListener("resize",()=>{
  const s = Math.min(window.innerWidth, window.innerHeight) * 0.96;
  canvas.width = canvas.height = Math.max(400,Math.round(s));
});

// Initial resize trigger and animation start
window.dispatchEvent(new Event("resize"));
requestAnimationFrame(animate);
</script>
