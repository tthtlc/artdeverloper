---
layout: fullscreen
title: "Psychedelic Hypnotic Waves on Pulsing Hexapetal Flower"
tags:
  - graphics
---

<style>
        canvas {
            display: block;
            margin: 0 auto;
            border: 1px solid black;
        }
        .controls {
            margin-top: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .control-group {
            margin: 10px 0;
            display: flex;
            align-items: center;
        }
        .control-group label {
            margin-right: 10px;
        }
        .color-spectrum {
            margin: 10px 0;
            width: 300px;
        }
        input[type="range"] {
            width: 200px;
        }
        .value-label {
            margin-left: 10px;
            font-weight: bold;
        }
</style>
<canvas id="psychedelicCanvas" width="700" height="700"></canvas>

<script>
const canvas = document.getElementById('psychedelicCanvas');
const ctx = canvas.getContext('2d');

const w = canvas.width, h = canvas.height;
const cx = w / 2, cy = h / 2;

const petals = 6;  // symmetry
const nRipples = 7;
const t0 = Date.now();

function hsl(a, s, l, apha=1) {
  return `hsla(${a},${s}%,${l}%,${apha})`;
}

// Draw a single flower petal with waves
function drawPetal(angle, pulse, t) {
  const steps = 240;
  ctx.save();
  ctx.rotate(angle);

  ctx.beginPath();
  for (let i = 0; i <= steps; ++i) {
    const theta = Math.PI * i/steps;
    // Main shape radius curve for flower
    let base = 130 + 32*Math.sin(theta) - 7*Math.sin(4*theta);
    // Modulate petal width with pulse
    let petalWidth = 115 + 25 * Math.sin(theta + pulse);
    // Hypnotic wave: nested sine + ripple + time
    let ripple = 0;
    for (let k=1; k <= nRipples; ++k) {
      ripple += (9/k) * Math.sin(
        k*theta*2 + 
        k*0.7*Math.sin(1.7*t + k) + 
        2*pulse + k
      );
    }
    const totalR = base + ripple + 11*Math.sin(5*theta + t/3);

    // Warped polar flower
    const x = Math.cos(theta-Math.PI/2) * (totalR)
             + Math.cos(theta) * petalWidth * 0.03;
    const y = Math.sin(theta-Math.PI/2) * (totalR)
             + Math.sin(theta) * petalWidth * 0.12;
    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.closePath();

  // Color: iridescent shifting along petal
  const hue = ((angle/(2*Math.PI)) * 360 + 200 + 32*pulse) % 360;
  const light = 55 + 25*Math.sin(2*pulse+Math.sin(t)+angle);
  ctx.strokeStyle = hsl(hue, 98, light, 0.85);
  ctx.shadowBlur = 16; ctx.shadowColor = hsl(hue,100,84);

  ctx.lineWidth = 3.2 + 2.2*Math.abs(Math.sin(pulse * 1.8));
  ctx.stroke();
  ctx.shadowBlur = 0;
  ctx.restore();
}

function drawWavefield(t) {
  // Animate a swirling plasma-like background
  const image = ctx.createImageData(w, h);
  const data = image.data;
  const c1 = [25, 14, 80], c2 = [250, 239, 37];

  for (let y = 0; y < h; y += 2) {
    for (let x = 0; x < w; x += 2) {
      // Normalize to center [-1,1]
      const nx = (x-cx) / (w/2), ny = (y-cy) / (h/2);
      const r = Math.sqrt(nx*nx + ny*ny);
      const a = Math.atan2(ny, nx);

      // Wavy field with swirling motion
      const val = 0.4 * Math.sin(9*r - 3*a + t) +
                  0.5 * Math.cos(14*r*r + a*5 + t * 0.71) +
                  0.19 * Math.sin(12*a + t*1.5 + r*11);
      // Map to [0,1]
      const f = (val + 1.5) / 3;

      // Color lerp between two colors
      const R = c1[0] + (c2[0] - c1[0]) * f;
      const G = c1[1] + (c2[1] - c1[1]) * f;
      const B = c1[2] + (c2[2] - c1[2]) * f;
      // Draw as 2x2 "pixel" for speed
      for (let dy=0; dy<2; ++dy) for (let dx=0; dx<2; ++dx) {
        const idx = 4*((y+dy)*w+(x+dx));
        data[idx] = R;
        data[idx+1] = G;
        data[idx+2] = B;
        data[idx+3] = 250;
      }
    }
  }

  ctx.putImageData(image,0,0);
}

function draw(t) {
  ctx.save();
  drawWavefield(t);

  // Slight zoom pulsing and rotation
  const pulse = 0.5 + 0.5 * Math.sin(t/2.5);
  ctx.translate(cx, cy);
  ctx.scale(1 + 0.07*pulse, 1 + 0.07*pulse);
  ctx.rotate(0.18 * Math.sin(t/1.4));

  // Draw overlapping rotating hypnotic petals
  for (let i = 0; i < petals; ++i) {
    const angle = ((2 * Math.PI) / petals) * i + 0.16 * Math.sin(t/1.7 + i);
    drawPetal(angle, pulse + 0.18*i + 0.5*Math.sin(3*t + i), t);
  }
  ctx.restore();

  // Overlay radiating pulse lines for more vibe
  ctx.save();
  ctx.translate(cx, cy);
  ctx.globalAlpha = 0.33+0.13*Math.sin(t*1.5);

  for (let i = 0; i < 40; ++i) {
    const a = (2*Math.PI)*i/40 + 0.4*Math.sin(t + i);
    ctx.save();
    ctx.rotate(a);
    ctx.strokeStyle = hsl(190 + 120*Math.sin(t+a), 80, 60+25*Math.sin(a-t), 0.34);
    ctx.beginPath();
    const r1 = 170 + 60*Math.sin(t+a*2);
    const r2 = 310 + 40*Math.cos(t*0.7 - a*2);
    ctx.moveTo(r1, 0);
    ctx.lineTo(r2, 0);
    ctx.lineWidth = 1.3 + 0.8*Math.abs(Math.cos(a*3+t));
    ctx.stroke();
    ctx.restore();
  }
  ctx.restore();
}

function animate() {
  const t = (Date.now() - t0) / 900;
  draw(t);
  requestAnimationFrame(animate);
}

animate();
</script>
