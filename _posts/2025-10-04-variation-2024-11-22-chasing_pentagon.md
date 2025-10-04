---
layout: fullscreen
title: "Hypnotic Sinusoidal Spirals"
tags:
  - graphics
---

<canvas id="hypnoSpiralCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('hypnoSpiralCanvas');
const ctx = canvas.getContext('2d');
canvas.style.background = '#120b23';

const W = canvas.width, H = canvas.height;

// Particle parameters
const NUM_ARMS = 7;
const PARTICLES_PER_ARM = 70;

const BASE_RADIUS = 60;
const ARM_STEP = 6;
const PARTICLE_STEP = 7.2;

const COLORS = [
  '#ff43d4', '#36fff1', '#f9ff41', 
  '#ff3e32', '#5fd579', '#f29fef',
  '#80caff', '#f7b940',
];

// Helper: get spiral point position
function getSpiralPoint(armIdx, pIdx, time) {
  // Evolving the spiral parameters with time for animation
  const dynamicRadius = BASE_RADIUS + pIdx * ARM_STEP + 20 * Math.sin(0.8 * time + pIdx/10 + armIdx);
  const angle =
    armIdx * (2 * Math.PI / NUM_ARMS)
    + pIdx * (Math.PI / (PARTICLES_PER_ARM/1.5))
    + 0.7 * Math.sin(time * 0.6 + armIdx + pIdx*0.11)
    + 0.12 * Math.cos(time + armIdx * 1.3 + pIdx * 0.9);

  // Sinusoidal wave modulation per arm
  const offsetR =
    15 * Math.sin(time*1.7 + pIdx*0.15 + armIdx * 0.4);

  return {
    x: W/2 + (dynamicRadius + offsetR) * Math.cos(angle),
    y: H/2 + (dynamicRadius + offsetR) * Math.sin(angle)
  };
}

// Animation loop
let t = 0;
function draw() {
  ctx.globalAlpha = 0.18;
  ctx.fillStyle = "#120b23";
  ctx.fillRect(0,0,W,H);
  ctx.globalAlpha = 1;

  // Draw the spiral arms
  for (let arm = 0; arm < NUM_ARMS; arm++) {
    for (let pi = 0; pi < PARTICLES_PER_ARM; pi++) {
      const pos = getSpiralPoint(arm, pi, t);

      // Calculate pulsation for width/size
      const pulse = 2 + Math.sin(t*2 + pi*0.23 + arm*0.7) * 2;
      const hueIdx = (arm + pi) % COLORS.length;
      ctx.save();
      ctx.beginPath();
      ctx.arc(pos.x, pos.y, 3.5 + pulse, 0, Math.PI*2);
      ctx.shadowColor = COLORS[hueIdx];
      ctx.shadowBlur = 18 + 8 * Math.sin(t + pi * 0.5);
      ctx.fillStyle = COLORS[hueIdx];
      ctx.globalAlpha = 0.56 + 0.39 * Math.sin(t*1.4 + pi * 0.15 + arm*0.6);

      // Draw pulsing "bubbles"
      ctx.fill();
      ctx.restore();

      // Draw small moving "trails" (psychedelic sparkles)
      if (pi < PARTICLES_PER_ARM-4 && pi % 4 === 0) {
        for (let j = 1; j < 5; j++) {
          const next = getSpiralPoint(arm, pi+j, t);
          ctx.save();
          ctx.globalAlpha = 0.1+0.1*j;
          ctx.strokeStyle = COLORS[(hueIdx+1)%COLORS.length];
          ctx.lineWidth = 2.6 - 0.5 * j;
          ctx.beginPath();
          ctx.moveTo(pos.x, pos.y);
          ctx.lineTo(next.x, next.y);
          ctx.stroke();
          ctx.restore();
        }
      }
    }
  }

  // Central cycling mandala
  ctx.save();
  ctx.translate(W/2, H/2);
  for(let i=0; i<NUM_ARMS; i++) {
    ctx.save();
    ctx.rotate(i*2*Math.PI/NUM_ARMS + Math.sin(t*0.93 + i));
    ctx.beginPath();
    for (let a = 0; a < Math.PI*2; a += Math.PI/8) {
      let r = 25 + 9*Math.sin(a*4 + t*2 + i*1.32);
      ctx.lineTo(Math.cos(a)*r, Math.sin(a)*r);
    }
    ctx.closePath();
    ctx.strokeStyle = COLORS[(i+Math.floor(t*1.7))%COLORS.length];
    ctx.lineWidth = 2.5 + Math.sin(t + i);
    ctx.globalAlpha = 0.44 + 0.18 * Math.cos(t*1.24 + i);
    ctx.shadowColor = COLORS[(i+2)%COLORS.length];
    ctx.shadowBlur = 12;
    ctx.stroke();
    ctx.restore();
  }
  ctx.restore();

  t += 0.018;
  requestAnimationFrame(draw);
}

draw();
</script>
