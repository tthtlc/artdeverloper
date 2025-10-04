---
layout: fullscreen
title: Hypnotic Flower Pulse with Radiant Cycling Petals
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Hypnotic Flower Pulse with Radiant Cycling Petals</title>
  <style>
    #container {
      position: relative;
      width: 500px;
      height: 500px;
      margin: auto;
    }
    canvas {
      position: absolute;
      top: 0;
      left: 0;
      border: 1px solid #191919;
      display: block;
      background: #101015;
    }
    #controls {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 20px;
    }
    #controls label {
      margin: 10px 0;
      color: #eee;
      font-size: 16px;
      font-family: monospace;
      letter-spacing: 1px;
    }
    body {
      background: #111115;
    }
  </style>
</head>
<body>
  <div id="container">
    <canvas id="canvas" width="500" height="500"></canvas>
  </div>
  
  <div id="controls">
    <label>
      Petals: <input type="range" id="petalSlider" min="5" max="20" step="1" value="8">
    </label>
    <label>
      Pulse: <input type="range" id="pulseSlider" min="1" max="12" step="0.1" value="4">
    </label>
    <label>
      Speed: <input type="range" id="speedSlider" min="0.001" max="0.05" step="0.001" value="0.017">
    </label>
  </div>
  
<script>
  document.addEventListener("contextmenu", function(event) { event.preventDefault(); });

  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');
  let width = canvas.width;
  let height = canvas.height;
  let centerX = width/2;
  let centerY = height/2;
  
  // Controls
  let numPetals = parseInt(document.getElementById('petalSlider').value);
  let pulse = parseFloat(document.getElementById('pulseSlider').value);
  let speed = parseFloat(document.getElementById('speedSlider').value);
  
  document.getElementById('petalSlider').addEventListener('input', function() {
    numPetals = parseInt(this.value);
  });
  document.getElementById('pulseSlider').addEventListener('input', function() {
    pulse = parseFloat(this.value);
  });
  document.getElementById('speedSlider').addEventListener('input', function() {
    speed = parseFloat(this.value);
  });

  // State
  let t = 0;
  let history = [];
  const maxTrailLen = 90;
  
  // Helper: HSV to RGB
  function hsv2rgb(h, s, v) {
    let r, g, b, i, f, p, q, t2;
    i = Math.floor(h * 6);
    f = h * 6 - i;
    p = v * (1 - s);
    q = v * (1 - f * s);
    t2 = v * (1 - (1 - f) * s);
    switch (i % 6) {
      case 0: r = v, g = t2, b = p; break;
      case 1: r = q, g = v, b = p; break;
      case 2: r = p, g = v, b = t2; break;
      case 3: r = p, g = q, b = v; break;
      case 4: r = t2, g = p, b = v; break;
      case 5: r = v, g = p, b = q; break;
    }
    return "rgb(" +
      Math.round(r * 255) + "," +
      Math.round(g * 255) + "," +
      Math.round(b * 255) + ")";
  }
  
  // Main animation
  function draw() {
    ctx.globalAlpha = 0.14;
    ctx.fillStyle = "#101015";
    ctx.fillRect(0,0,width,height); // Fade old trails
    ctx.globalAlpha = 1.0;

    // Parameters
    const baseRadius = 90 + 60*Math.sin(t*0.17);
    const petalRadius = 130 + 10 * Math.cos(t*0.09);
    const corePulse = 22 + 16*Math.sin(t*1.73);

    // --- Draw pulsing flower petals ---
    ctx.save();
    ctx.translate(centerX, centerY);
    for(let i=0;i<numPetals;++i) {
      let theta = (Math.PI*2*i)/numPetals + t*0.4;
      let localPulse = 1 + 0.20*Math.sin(pulse*theta + t*2.1 - Math.cos(t*0.7+i));
      let r1 = baseRadius * localPulse;
      let r2 = petalRadius * localPulse;
      let x1 = Math.cos(theta)*r1;
      let y1 = Math.sin(theta)*r1;
      let x2 = Math.cos(theta+Math.PI/numPetals)*r2;
      let y2 = Math.sin(theta+Math.PI/numPetals)*r2;
      let petalColor = hsv2rgb(((t*0.055 + i/numPetals)%1), 0.8, 0.98);
      ctx.beginPath();
      ctx.moveTo(0,0);
      ctx.quadraticCurveTo(x2,y2,x1,y1);
      ctx.arc(0,0,r1,theta,theta+Math.PI*2/numPetals,false);
      ctx.closePath();
      ctx.fillStyle = petalColor;
      ctx.globalAlpha = 0.74;
      ctx.shadowColor = hsv2rgb(((t*0.055 + i/numPetals+.2)%1),0.85,1.0);
      ctx.shadowBlur = 20+12*Math.sin(theta*2+t*1.2);
      ctx.fill();
      ctx.globalAlpha = 1.0;
      ctx.shadowBlur = 0;
    }
    ctx.restore();
    
    // --- Draw inner rotating core ---
    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.beginPath();
    for(let i=0;i<=120;++i) {
      let angle = (Math.PI*2*i)/120;
      let r = corePulse * (1.05 + 0.14*Math.sin(i*0.34 + t*3 + Math.sin(t*0.014)));
      if(i===0) ctx.moveTo(Math.cos(angle)*r, Math.sin(angle)*r);
      else ctx.lineTo(Math.cos(angle)*r, Math.sin(angle)*r);
    }
    ctx.closePath();
    ctx.globalAlpha = 0.71;
    ctx.fillStyle = "#f3ffb0";
    ctx.shadowColor = "#e8d3ff";
    ctx.shadowBlur = 15+10*Math.sin(t*1.872);
    ctx.fill();
    ctx.globalAlpha = 1.0;
    ctx.restore();

    // --- Draw radiating cycling points (trail points) ---
    let trailRadius = baseRadius*1.04 + 19*Math.sin(t*0.5);
    // Leave colorful trails
    let px = centerX + Math.cos(t*speed*49 + Math.sin(t*0.3))*trailRadius;
    let py = centerY + Math.sin(t*speed*47 + Math.cos(t*0.31))*trailRadius;
    history.push({x: px, y: py, phase: t});
    if(history.length > maxTrailLen) history.shift();

    for(let k=1;k<history.length;++k) {
      let p0 = history[k-1];
      let p1 = history[k];
      let hue = ((p1.phase*0.1 + k*0.0125)%1);
      ctx.beginPath();
      ctx.moveTo(p0.x, p0.y);
      ctx.lineTo(p1.x, p1.y);
      ctx.strokeStyle = hsv2rgb(hue, 1, 0.95);
      ctx.globalAlpha = 0.07 + 0.13*k/history.length;
      ctx.lineWidth = 2.5 - 1.2*k/history.length;
      ctx.shadowColor = hsv2rgb((hue+0.12)%1, 1, 1);
      ctx.shadowBlur = 14*Math.pow(1-k/history.length,0.55);
      ctx.stroke();
      ctx.shadowBlur = 0;
    }
    ctx.globalAlpha = 1;

    // --- Draw the frontmost glowing orb ---
    let orbX = px, orbY = py;
    ctx.beginPath();
    ctx.arc(orbX, orbY, 6+3*Math.sin(t*2.9), 0, 2*Math.PI);
    ctx.fillStyle = hsv2rgb((t*0.1)%1, 0.95, 1);
    ctx.shadowColor = hsv2rgb((t*0.12+0.4)%1, 1, 1);
    ctx.shadowBlur = 22+7*Math.abs(Math.sin(t));
    ctx.globalAlpha = 0.8;
    ctx.fill();
    ctx.shadowBlur = 0; ctx.globalAlpha = 1;

    t += speed*2.1;

    requestAnimationFrame(draw);
  }
  
  // Initial fill
  ctx.fillStyle = "#101015";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  draw();
</script>
</body>
</html>
```
