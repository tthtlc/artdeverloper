---
layout: fullscreen
title: "Waves of Pulsing Neon Circles & Spiral Drift"
tags:
  - graphics
---

<canvas id="neonSpiralCanvas" width="700" height="700"></canvas>
<script>
// ===== PARAMETERS =====
const canvas = document.getElementById('neonSpiralCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;
const cx = W/2;
const cy = H/2;

const NUM_WAVES = 7;
const CIRCLES_PER_WAVE = 28;
const SPIRAL_RADIUS = 220;
const CIRCLE_MIN = 12;
const CIRCLE_MAX = 35;

const palette = [
  [27, 245, 218],
  [228, 16, 116],
  [255, 222, 89],
  [38, 0, 255],
  [255, 41, 0],
  [0, 255, 115],
  [247, 108, 255]
];
  
function lerp(a, b, t) { return a + (b - a) * t; }

function hslaCycle(hue, sat=70, light=53, a=1) {
  // hue in degrees
  return `hsla(${Math.round(hue)},${sat}%,${light}%,${a})`;
}

// ===== WAVE DATA =====
let t = 0;

function draw() {
  ctx.globalCompositeOperation = 'lighter';
  // Semi-transparent fadeout for echo trails:
  ctx.globalAlpha = 0.22;
  ctx.fillStyle = "black";
  ctx.fillRect(0, 0, W, H);
  ctx.globalAlpha = 1;

  // Animate spiral's rotation for added motion
  const spiralPhase = t * 0.00018;

  for(let w=0; w<NUM_WAVES; w++) {
    // Each wave has own frequency/phase/color drift
    const colorIdx = w % palette.length;
    const [r, g, b] = palette[colorIdx];
    const phase = spiralPhase + w * Math.PI * 2 / NUM_WAVES;

    for(let i=0; i<CIRCLES_PER_WAVE; i++) {
      // Spiral coordinates with radial drifted by Lissajous oscillation
      const t0 = (i + t*0.027) / (CIRCLES_PER_WAVE + 5);
      const angle = t0 * Math.PI*2 * 2 + phase;
      const baseRad = lerp(SPIRAL_RADIUS * 0.5, SPIRAL_RADIUS, t0);

      // Center for this circle
      let x = cx + baseRad * Math.cos(angle + 0.11 * Math.sin(w+angle+t*0.0027));
      let y = cy + baseRad * Math.sin(angle + 0.17 * Math.cos(i+t*0.0041));

      // Radial breathing via double sine
      const breathe = 0.6 + 0.4 * Math.sin(phase*0.7 + angle*2 + t*0.018 + w*3);
      const modCircle = lerp(CIRCLE_MIN, CIRCLE_MAX, breathe);

      // Pulsating color: hue wheel + palette overlay
      const hue = ((angle * 40) + t*0.13 + w*41 + 360) % 360;
      ctx.save();
      ctx.beginPath();
      ctx.arc(x, y, modCircle*0.82 + 0.7*(Math.sin(i+w+t*0.02)*modCircle), 0, Math.PI*2);

      // Neon glow!
      ctx.shadowColor = `rgba(${r},${g},${b},0.7)`;
      ctx.shadowBlur = 15 + 4*modCircle;
      ctx.fillStyle = hslaCycle(hue, 75, lerp(35, 68, breathe), 0.46 + 0.41*Math.sin(i+t*0.021+w));
      ctx.fill();

      // Inner core "blink"
      ctx.shadowBlur = 2 + 5*breathe;
      ctx.shadowColor = `rgb(${r},${g},${b})`;
      ctx.beginPath();
      ctx.arc(x, y, modCircle*0.46 + 1.8*Math.sin(angle*3.7 + t*0.011), 0, Math.PI*2);
      ctx.fillStyle = `rgba(${r},${g},${b},0.97)`;
      ctx.globalAlpha = 0.63 + 0.34*Math.cos(angle+t*0.018-w);
      ctx.fill();

      ctx.globalAlpha = 1;
      ctx.restore();
    }
  }
  t += 2.25;
  requestAnimationFrame(draw);
}

// Responsive resize
function resize() {
  const dpr = window.devicePixelRatio || 1;
  const size = Math.min(window.innerWidth, window.innerHeight, 900);
  canvas.width = canvas.height = size*dpr;
  canvas.style.width = canvas.style.height = size+"px";
}

resize();
window.addEventListener("resize", () => {resize(); draw();});

// Start animation when ready
draw();
</script>