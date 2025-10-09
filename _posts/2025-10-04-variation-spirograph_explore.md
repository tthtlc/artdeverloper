---
layout: fullscreen
title: Psychedelic Harmonic Lissajous Waves with Color Trails
tags:
  - graphics
---

<style>
    #container {
      position: relative;
      width: 500px;
      height: 500px;
      margin: auto;
      margin-bottom: 30px;
    }
    canvas {
      left: 0;
      top: 0;
      display: block;
      border: 1px solid #161616;
      z-index: 1;
    }
    #controls {
      display: flex;
      flex-direction: row;
      justify-content: center;
      gap: 20px;
      margin-top: 0;
    }
    #controls label {
      font-family: monospace;
      font-size: 15px;
      color: #f0f0f0;
    }
    input[type=range] {
      width: 120px;
    }
    select {
      font-family: monospace;
      font-size: 14px;
    }
</style>

<div id="container">
	<canvas id="canvas" width="500" height="500"></canvas>
	<canvas id="trailCanvas" width="500" height="500"></canvas>
</div>
<div id="controls">
    <label>
      <span>freqA:</span> <input type="range" id="freqASlider" min="1" max="8" step="1" value="3">
    </label>
    <label>
      <span>freqB:</span> <input type="range" id="freqBSlider" min="1" max="8" step="1" value="2">
    </label>
    <label>
      <span>phaseDiff:</span> <input type="range" id="phaseSlider" min="0" max="628" step="1" value="200">
    </label>
    <label>
      <span>waveMode:</span>
      <select id="waveModeSelect">
        <option value="lissajous">Lissajous</option>
        <option value="spiro">Spirograph</option>
        <option value="ripple">Ripple</option>
      </select>
    </label>
</div>
<script>
  document.addEventListener("contextmenu", e => e.preventDefault());

  const canvas = document.getElementById('canvas');
  const trailCanvas = document.getElementById('trailCanvas');
  const ctx = canvas.getContext('2d');
  const trailCtx = trailCanvas.getContext('2d');
  const w = canvas.width, h = canvas.height;

  // Controls
  let freqA = +document.getElementById('freqASlider').value;
  let freqB = +document.getElementById('freqBSlider').value;
  let phaseDiff = +document.getElementById('phaseSlider').value / 100;
  let waveMode = document.getElementById('waveModeSelect').value;

  document.getElementById('freqASlider').addEventListener('input', function() {
    freqA = +this.value;
    clearTrail();
  });
  document.getElementById('freqBSlider').addEventListener('input', function() {
    freqB = +this.value;
    clearTrail();
  });
  document.getElementById('phaseSlider').addEventListener('input', function() {
    phaseDiff = +this.value / 100;
    clearTrail();
  });
  document.getElementById('waveModeSelect').addEventListener('change', function() {
    waveMode = this.value;
    clearTrail();
  });
  
  function clearTrail() {
    trailCtx.clearRect(0, 0, w, h);
    lastPt = undefined;
    colorTick = 0;
  }

  // Animation vars
  let t = 0, lastPt = undefined, colorTick = 0;
  const speed = 0.03;

  // Lissajous params
  const size = 180;
  const center = [w/2, h/2];

  function getColor(theta) {
    // Psychedelic color cycling
    // theta: radians (use t or position to vary)
    const r = 180 + 70*Math.sin(theta*2 + colorTick*0.008);
    const g = 170 + 70*Math.sin(theta + 2 + colorTick*0.01);
    const b = 200 + 55*Math.sin(theta*2 + colorTick*0.015 - 2);
    return `rgb(${r|0},${g|0},${b|0})`;
  }

  function lissajousPoint(t, freqA, freqB, phase) {
    const x = center[0] + size * Math.sin(freqA * t + phase);
    const y = center[1] + size * Math.sin(freqB * t);
    return [x, y];
  }

  function spirographPoint(t, freqA, freqB, phase) {
    // Rose curve variation
    const k = freqA / freqB;
    const r = size * Math.cos(k * t + phase);
    const x = center[0] + r * Math.cos(t);
    const y = center[1] + r * Math.sin(t);
    return [x, y];
  }

  function ripplePoint(t, freqA, freqB, phase) {
    // Expanding evolving waves
    const r = size + 45 * Math.sin(freqB * t + phase + Math.cos(freqA*t)/2);
    const x = center[0] + r * Math.cos(t*0.7 + Math.sin(freqB*t)*0.6);
    const y = center[1] + r * Math.sin(t*0.7 + Math.cos(freqA*t)*0.6);
    return [x, y];
  }

  function draw() {
    // Faint transparency layer to fade old trails
    trailCtx.fillStyle = "rgba(8,8,16,0.08)";
    trailCtx.fillRect(0, 0, w, h);

    ctx.clearRect(0, 0, w, h);

    // Draw main evolving waveform curve -- for preview
    ctx.save();
    ctx.globalAlpha = 0.45;
    ctx.setLineDash([4, 10]);
    ctx.lineWidth = 1.5;
    ctx.strokeStyle = "#666";
    ctx.beginPath();
    let prev;
    for(let tt=0; tt < 2*Math.PI; tt += 0.002) {
      let p;
      if (waveMode === "lissajous")
        p = lissajousPoint(tt, freqA, freqB, phaseDiff);
      else if (waveMode === "spiro")
        p = spirographPoint(tt, freqA, freqB, phaseDiff);
      else
        p = ripplePoint(tt, freqA, freqB, phaseDiff);
      if (!prev) ctx.moveTo(p[0], p[1]);
      else ctx.lineTo(p[0], p[1]);
      prev = p;
    }
    ctx.stroke();
    ctx.restore();

    // Animate moving "tracer" with rainbow trail
    let pt;
    if (waveMode === "lissajous")
      pt = lissajousPoint(t, freqA, freqB, phaseDiff);
    else if (waveMode === "spiro")
      pt = spirographPoint(t, freqA, freqB, phaseDiff);
    else
      pt = ripplePoint(t, freqA, freqB, phaseDiff);

    if (lastPt) {
      trailCtx.strokeStyle = getColor(t);
      trailCtx.lineWidth = 2.8+1.5*Math.sin(t/6);
      trailCtx.beginPath();
      trailCtx.moveTo(lastPt[0], lastPt[1]);
      trailCtx.lineTo(pt[0], pt[1]);
      trailCtx.stroke();
    }
    lastPt = pt;

    // Draw current tracer
    ctx.save();
    ctx.shadowBlur = 15;
    ctx.shadowColor = getColor(t+10);
    ctx.beginPath();
    ctx.arc(pt[0], pt[1], 10 + 2*Math.sin(t/3), 0, 2*Math.PI);
    ctx.fillStyle = getColor(t+23);
    ctx.fill();
    ctx.restore();

    colorTick += 1;
    t += speed;

    requestAnimationFrame(draw);
  }

  clearTrail();
  draw();
</script>
