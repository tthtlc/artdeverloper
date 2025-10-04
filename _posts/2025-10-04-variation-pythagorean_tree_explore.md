---
layout: fullscreen
title: Pulsating Lissajous Wave Petals
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Pulsating Lissajous Wave Petals</title>
  <style>
    html, body {
      margin: 0;
      padding: 0;
      height: 100%;
      background: #07001a;
    }
    body {
      height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: start;
      font-family: Arial, sans-serif;
      background: radial-gradient(ellipse at center, #130031 0%, #07001a 100%);
    }
    .controls {
      margin-top: 24px;
      margin-bottom: 18px;
      background: rgba(10,6,34, 0.50);
      box-shadow: 0 2px 24px #2225, 0 1px 8px #13112c75;
      border-radius: 16px;
      padding: 16px 32px;
      display: flex;
      gap: 40px;
      align-items: center;
      color: #fafaff;
      font-size: 1rem;
    }
    .controls label {
      margin-right: 8px;
      font-weight: 600;
      letter-spacing: 0.02em;
    }
    canvas {
      box-shadow: 0 6px 32px #13003155, 0 1px 10px #05020999;
      border-radius: 18px;
      border: 2px solid #24175a;
      display: block;
    }
  </style>
</head>
<body>
  <div class="controls" id="controls">
    <div>
      <label for="freqX">X Frequency:</label>
      <input type="range" min="1" max="12" step="1" value="3" id="freqX" oninput="updateFreqX(this.value)">
      <span id="freqXval">3</span>
    </div>
    <div>
      <label for="freqY">Y Frequency:</label>
      <input type="range" min="1" max="12" step="1" value="5" id="freqY" oninput="updateFreqY(this.value)">
      <span id="freqYval">5</span>
    </div>
    <div>
      <label for="petals">Petal Count:</label>
      <input type="range" min="2" max="16" step="1" value="8" id="petals" oninput="updatePetals(this.value)">
      <span id="petalsVal">8</span>
    </div>
    <div>
      <label for="saturation">Saturation:</label>
      <input type="range" min="30" max="100" step="1" value="80" id="saturation" oninput="updateSaturation(this.value)">
      <span id="saturationVal">80</span>
    </div>
  </div>
  <canvas id="canv" width="720" height="640"></canvas>

  <script>
    // Animation State
    let freqX = 3;
    let freqY = 5;
    let petalCount = 8;
    let saturation = 80;

    function updateFreqX(v) {
      freqX = parseInt(v);
      document.getElementById("freqXval").textContent = v;
    }
    function updateFreqY(v) {
      freqY = parseInt(v);
      document.getElementById("freqYval").textContent = v;
    }
    function updatePetals(v) {
      petalCount = parseInt(v);
      document.getElementById("petalsVal").textContent = v;
    }
    function updateSaturation(v) {
      saturation = parseInt(v);
      document.getElementById("saturationVal").textContent = v;
    }

    // Helper: HSL with alpha
    function hsl(h, s, l, a=1) {
      return `hsla(${h},${s}%,${l}%,${a})`;
    }

    // --- Main Animation ---
    const canvas = document.getElementById("canv");
    const ctx = canvas.getContext("2d");

    function drawLissajousPetal(cx, cy, r, fx, fy, phase, time, petalIdx, totalPetals) {
      // Generate an animated Lissajous petal
      ctx.save();
      ctx.translate(cx, cy);
      ctx.rotate((2 * Math.PI * petalIdx) / totalPetals);

      ctx.beginPath();
      let nPoints = 200;
      for (let i = 0; i <= nPoints; i++) {
        let t = i / nPoints * 2 * Math.PI;
        // Pulsating radius along the lissajous curve
        let base = r * (0.55 + 0.28 * Math.sin(time*1.3 + petalIdx + 2 * t));
        let x = base * Math.sin(fx * t + phase + 0.31 * Math.cos(time + t + petalIdx));
        let y = base * Math.sin(fy * t + 0.7*phase + 0.53 * Math.sin(time*1.7 + t - petalIdx/2));

        // Give depth with z pulsation and vertical warp
        let z = Math.sin(t + time + petalIdx) * 22;
        x += z * Math.cos(t + petalIdx) * 0.4;
        y += z * Math.sin(t + petalIdx) * 0.2;

        if (i == 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();

      // Colorful gradient fill based on angle/petal, modulate hue
      let hue = ((360/petalCount)*petalIdx + 200 + 50*Math.sin(time*0.7+petalIdx)) % 360;
      let light = 53 + 13 * Math.sin(time*0.8 + petalIdx);

      // Layered fills for depth
      let grad = ctx.createRadialGradient(0, 0, 0, 0, 0, r*0.8);
      grad.addColorStop(0.0, hsl(hue, saturation, 65, 0.61));
      grad.addColorStop(0.4, hsl(hue, saturation, light, 0.95));
      grad.addColorStop(1.0, hsl((hue+60)%360, saturation-40, 23, 0.06));

      ctx.fillStyle = grad;
      ctx.shadowColor = hsl(hue, 74, 36, 0.17);
      ctx.shadowBlur = 21 + 14 * Math.sin(petalIdx + time);

      ctx.globalAlpha = 0.80 + 0.07*Math.sin(petalIdx*2+time*1.8);
      ctx.fill();

      // Optionally draw highlight outline
      ctx.lineWidth = 1.55 + 0.7*Math.sin(time + petalIdx);
      ctx.strokeStyle = hsl((hue+28)%360, saturation+8, 97, 0.22);
      ctx.globalAlpha = 0.7;
      ctx.stroke();

      ctx.restore();
    }

    function animate() {
      // Resize canvas to window (centered, but limited aspect)
      let size = Math.min(window.innerWidth * 0.95, window.innerHeight * 0.82);
      size = Math.max(400, Math.min(size, 880));
      canvas.width = size;
      canvas.height = size * 0.89;

      let cx = canvas.width / 2, cy = canvas.height / 2.05;
      let radius = size * 0.32;

      // Background psychedelic fading blobs
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      let T = performance.now() * 0.001;
      for (let j = 0; j < 4; j++) {
        let btheta = T*0.22 + Math.PI*2*j/4;
        let bx = cx + Math.cos(btheta + Math.sin(T*0.77+j))*radius*1.23;
        let by = cy + Math.sin(btheta + Math.cos(T*0.58-j))*radius*1.14;
        let brad = radius * (0.88 + 0.18*Math.sin(T*1.5+j));
        let bgrad = ctx.createRadialGradient(bx, by, 0, bx, by, brad);
        let bhue = ((T*49)+(j*103))%360;
        bgrad.addColorStop(0.0, hsl(bhue, 68, 59, 0.12));
        bgrad.addColorStop(0.4, hsl((bhue+40)%360, 44, 43, 0.09));
        bgrad.addColorStop(1.0, hsl((bhue+80)%360, 50, 12, 0.01));
        ctx.beginPath();
        ctx.arc(bx, by, brad, 0, Math.PI*2);
        ctx.fillStyle = bgrad;
        ctx.globalAlpha = 1;
        ctx.fill();
      }

      // Draw each petal along a circle
      let phase = T * 0.8;
      for (let i = 0; i < petalCount; i++) {
        drawLissajousPetal(cx, cy, radius,
          freqX,
          freqY,
          phase + i*0.8 + Math.sin(T*0.6+i)*0.23,
          T,
          i,
          petalCount
        );
      }

      // Central glowing core
      let coreGrad = ctx.createRadialGradient(cx, cy, 0, cx, cy, radius*0.21 + 3*Math.sin(T));
      coreGrad.addColorStop(0, hsl(62+14*Math.sin(T*0.7), saturation+20, 91, 0.48));
      coreGrad.addColorStop(0.6, hsl(342, 68, 88, 0.11));
      coreGrad.addColorStop(1, hsl(270, 36, 11, 0.04));
      ctx.beginPath();
      ctx.arc(cx, cy, radius*0.24 + 5*Math.sin(T*1.2), 0, Math.PI*2);
      ctx.fillStyle = coreGrad;
      ctx.globalAlpha = 1;
      ctx.fill();

      requestAnimationFrame(animate);
    }

    window.addEventListener('resize', () => {
      // Responsive resize on window change
      let size = Math.min(window.innerWidth * 0.95, window.innerHeight * 0.82);
      size = Math.max(400, Math.min(size, 880));
      canvas.width = size;
      canvas.height = size * 0.89;
    });

    // Initialize values
    document.getElementById("freqXval").textContent = freqX;
    document.getElementById("freqYval").textContent = freqY;
    document.getElementById("petalsVal").textContent = petalCount;
    document.getElementById("saturationVal").textContent = saturation;

    animate();
  </script>
</body>
</html>
```
