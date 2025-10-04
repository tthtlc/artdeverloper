---
layout: fullscreen
title: "Radiant Flower of Waves"
tags:
  - graphics
---

<canvas id="flowerCanvas" width="600" height="600"></canvas>
<script>

// Radiant Flower of Waves
const canvas = document.getElementById('flowerCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;

// Psychedelic color palette
const palette = [
  "#FF2E63","#08D9D6","#F8FF1F","#FA26A0","#3A0088","#D4FF00",
  "#F38181","#2BDE73","#6552FF","#FF8811","#FFABAB","#907AD6",
  "#FFC15E","#B6EAFA","#4895EF","#8AC926"
];

// Parameters for the flower waves
const petalCount = 8;
const baseRadius = 80;
const amplitude = 60;
const layers = 20;
const pointsPerPetal = 200;
let phase = 0;
let t = 0;

// Helper: get color from palette smoothly
function getGradientColor(a) {
  const n = palette.length;
  let f = a * n;
  let i = Math.floor(f);
  let j = (i + 1) % n;
  let mix = f - i;
  // Simple rgb lerp
  function hexToRgb(h){
    let r = parseInt(h.substr(1,2),16);
    let g = parseInt(h.substr(3,2),16);
    let b = parseInt(h.substr(5,2),16);
    return [r,g,b];
  }
  let c1 = hexToRgb(palette[i % n]);
  let c2 = hexToRgb(palette[j]);
  let r = Math.round(c1[0]*(1-mix)+c2[0]*mix);
  let g = Math.round(c1[1]*(1-mix)+c2[1]*mix);
  let b = Math.round(c1[2]*(1-mix)+c2[2]*mix);
  return `rgb(${r},${g},${b})`;
}

function draw() {
  ctx.clearRect(0,0,W,H);
  ctx.save();
  ctx.translate(W/2,H/2);

  // Subtle rotate and scale animation
  let rot = Math.sin(phase/1.8)*0.06;
  ctx.rotate(rot);
  let sc = 0.92 + Math.cos(phase/2.2)*0.07;
  ctx.scale(sc,sc);

  // Draw multiple flower layers, each evolving in time and petal count
  for (let l = 0; l < layers; l++) {
    let fract = l/layers;
    let layerPhase = phase + l*0.17;
    let dynamicPetals = petalCount + Math.sin(phase*0.25 + l*0.13)*2.5;
    let r0 = baseRadius + amplitude*fract + Math.sin(t*0.3 + l)*18;
    let waveAmp = amplitude*(0.6 + 0.36*Math.sin((phase*0.63+l)*1.1));

    ctx.beginPath();
    for (let i = 0; i <= pointsPerPetal; i++) {
      let a = (i/pointsPerPetal)*2*Math.PI;
      let theta = a * dynamicPetals;
      // Radial sine wave, oscillated over layer and time
      let r = r0 + waveAmp * Math.sin(theta + Math.sin(layerPhase)*1.5 + Math.cos(a + phase)*0.7);
      let px = r * Math.cos(a);
      let py = r * Math.sin(a);
      if(i===0) ctx.moveTo(px,py);
      else ctx.lineTo(px,py);
    }
    let col = getGradientColor((fract*0.65 + Math.sin(layerPhase)*0.28 + phase*0.07) % 1.0);
    ctx.strokeStyle = col;
    ctx.globalAlpha = 0.34 + 0.70*(1-fract)*0.43; // Fade out per layer
    ctx.lineWidth = 2.4 - 1.9*fract;
    ctx.shadowColor = col;
    ctx.shadowBlur = 12 + 24*fract;
    ctx.stroke();
    ctx.globalAlpha = 1.0;
  }

  ctx.restore();

  // Animate the background - a slowly cycling color fade
  let grad = ctx.createRadialGradient(W/2,H/2,0,W/2,H/2,W/1.9);
  grad.addColorStop(0, getGradientColor((0.22+phase*0.03)%1.0));
  grad.addColorStop(1, getGradientColor((0.7+Math.sin(phase*0.2)*0.24)%1.0));
  ctx.globalCompositeOperation = "destination-over";
  ctx.fillStyle = grad;
  ctx.fillRect(0,0,W,H);
  ctx.globalCompositeOperation = "source-over";

  phase += 0.023;
  t += 0.017;
  requestAnimationFrame(draw);
}

// Responsive resize
function resize() {
  let s = Math.min(window.innerWidth, window.innerHeight, 700);
  canvas.width = canvas.height = s;
}
window.addEventListener('resize', resize);
resize();

draw();

</script>