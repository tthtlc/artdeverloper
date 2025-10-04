---
layout: fullscreen
title: "Kaleidoscopic Particle Spirals"
tags:
  - graphics
---

<style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
    }
    body {
      width: 100vw;
      height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      background: #101020;
      overflow: hidden;
    }
    canvas {
      display: block;
      margin: 0 auto;
      background: transparent;
      border-radius: 20px;
      box-shadow: 0 6px 24px rgba(0,0,0,0.16);
    }
    .controls {
      display: flex;
      align-items: center;
      gap: 20px;
      margin-top: 12px;
      background: rgba(36, 38, 58, 0.84);
      border-radius: 12px;
      padding: 10px 16px;
      color: #eee;
      font-family: system-ui;
    }
    .label {
      font-weight: bold;
      margin-right: 6px;
    }
    .dropdown, .slider {
      display: flex;
      align-items: center;
    }
    .value {
      min-width:30px;
      margin-left:6px;
      font-family: monospace;
    }
</style>
<canvas id="spiralCanvas" width="640" height="640"></canvas>
<div class="controls">
    <div class="slider">
      <span class="label">Spirals:</span>
      <input type="range" id="armControl" min="3" max="16" value="6">
      <span id="armValue" class="value">6</span>
    </div>
    <div class="slider">
      <span class="label">Particles:</span>
      <input type="range" id="particleControl" min="200" max="900" value="480" step="10">
      <span id="particleValue" class="value">480</span>
    </div>
    <div class="dropdown">
      <span class="label">Palette:</span>
      <select id="paletteSelector">
        <option value="trippy">Trippy Rainbow</option>
        <option value="aether">Aether Fade</option>
        <option value="acid">Acid Pool</option>
        <option value="sunset">Psy Sunset</option>
      </select>
    </div>
    <div class="dropdown">
      <span class="label">BG:</span>
      <input type="color" id="bgColorPicker" value="#141428">
    </div>
  </div>

  <script>
    const w = 640, h = 640;
    const canvas = document.getElementById('spiralCanvas');
    canvas.width = w;
    canvas.height = h;
    const ctx = canvas.getContext('2d');

    // Palettes: arrays of color stops or functions
    const palettes = {
      trippy: [
        '#ff0080', '#ff8c00', '#40ffd7', '#00ff8c', '#00b0ff', '#ffff00', '#ee00ff', '#ff0080'
      ],
      aether: [
        '#bc6ff1', '#8372ff', '#23c4ed', '#62d2a2', '#c0f661', '#f4e285', '#f6b6c9', '#bc6ff1'
      ],
      acid: [
        '#fff201', '#ffc300', '#f79246', '#be1558', '#5f0a87', '#15f7fa', '#16ffbd', '#fff201'
      ],
      sunset: [
        '#fcb900', '#ff686b', '#be60a2', '#452597', '#4042cd', '#e93b81', '#fcb900'
      ]
    };

    // Controls
    let arms = 6;
    let nParticles = 480;
    let palette = palettes.trippy;
    let baseBg = "#141428";
    let bg = {r:20,g:20,b:40};

    const center = { x: w/2, y: h/2 };
    const spiralRadius = Math.min(w, h)/2.15;
    let time = 0;

    // UI interaction
    document.getElementById('armControl').addEventListener('input', e=>{
      arms = +e.target.value;
      document.getElementById('armValue').textContent = arms;
    });
    document.getElementById('particleControl').addEventListener('input', e=>{
      nParticles = +e.target.value;
      document.getElementById('particleValue').textContent = nParticles;
    });
    document.getElementById('paletteSelector').addEventListener('change', e=>{
      palette = palettes[e.target.value];
    });
    document.getElementById('bgColorPicker').addEventListener('input', e=>{
      baseBg = e.target.value;
      bg = hexToRgb(baseBg);
    });

    function hexToRgb(hex) {
      // Accepts #111 or #112233
      hex = hex.replace('#','');
      if (hex.length === 3) hex = hex.split('').map(c=>c+c).join('');
      const num = parseInt(hex,16);
      return {
        r: (num>>16)&255,
        g: (num>>8)&255,
        b: num&255
      };
    }

    // Morph the background color for subtle psychedelia
    function morphBgColor() {
      bg.r += (Math.random()-0.5)*2.2;
      bg.g += (Math.random()-0.5)*2.2;
      bg.b += (Math.random()-0.5)*2.5;
      // Clamp to 16..245 for pleasant backgrounds
      bg.r = Math.min(245,Math.max(16,bg.r));
      bg.g = Math.min(245,Math.max(16,bg.g));
      bg.b = Math.min(245,Math.max(16,bg.b));
      ctx.fillStyle = `rgb(${Math.round(bg.r)},${Math.round(bg.g)},${Math.round(bg.b)})`;
      ctx.fillRect(0, 0, w, h);
    }

    // Given a float t in [0,1], interpolate palette
    function lerpPalette(pal, t) {
      const n = pal.length-1;
      t = t%1;
      const scaled = t*n;
      const i = Math.floor(scaled);
      const f = scaled-i;
      const c1 = hexToRgb(pal[i]);
      const c2 = hexToRgb(pal[(i+1)%pal.length]);
      return `rgb(${Math.round(lerp(c1.r,c2.r,f))},${Math.round(lerp(c1.g,c2.g,f))},${Math.round(lerp(c1.b,c2.b,f))})`;
    }
    function lerp(a,b,t) { return a + (b-a)*t; }

    // Particle Spiral Drawing
    function drawKaleidoSpirals(dt) {
      morphBgColor();

      // Overlapping trailing transparent veil for motion trails
      ctx.globalAlpha = 0.24;
      ctx.fillStyle = 'rgba(18,20,34,0.8)';
      ctx.fillRect(0,0,w,h);
      ctx.globalAlpha = 1.0;

      // Modulate spiral "breathing"
      const spiralMod = Math.sin(time*0.48)*0.11 + 0.92;
      const tw = nParticles;

      for (let s=0; s<arms; ++s) {
        // Each 'arm' rotates at a slightly offset speed
        const armRot = s*Math.PI*2/arms;
        const armPhase = Math.sin(time*0.19 + s*1.9)*0.10;
        for (let i=0;i<tw;++i) {
          const t = i/tw;
          // Golden angle spiral for varied structure
          let ang = (t* arms + armPhase)*Math.PI*2 + armRot;
          // Animate modulations
          ang += Math.sin(time*0.34 + t*6.0 + s)*0.24;
          let r = spiralRadius*t*spiralMod;
          r *= 0.8 + 0.2*Math.sin(time*0.81 + t*4 + s*1.2);

          // Swirl effect
          ang += Math.sin(time*0.11 + r/70.0)*0.23;

          let x = center.x + r * Math.cos(ang);
          let y = center.y + r * Math.sin(ang);

          // Animate particle dither/wiggle
          let dx = Math.sin(time+t*6+s*1.7)*2.4;
          let dy = Math.cos(time*0.6+t*7-s*2.1)*2.1;
          x+=dx; y+=dy;

          // Depth pulse
          let sz = 2.3 + 2.2*Math.sin(time*0.7 + t*7 + s*1.1);
          if (i%30===0) sz *= 1.52; // Bright nodes

          // Color along palette via t, arm offset, time sin
          let colorRatio = t + 0.12*Math.sin(time*0.63 + s*0.7 + t*4.2);
          let col = lerpPalette(palette, (colorRatio + s/arms + time*0.06)%1);

          ctx.save();
          ctx.beginPath();
          ctx.arc(x, y, sz, 0, Math.PI*2);
          ctx.shadowColor = col;
          ctx.shadowBlur = 14 + 15*(sz/4);
          ctx.globalAlpha = 0.89 - 0.4*t + 0.02*Math.sin(time+s*2 + t*8);
          ctx.fillStyle = col;
          ctx.fill();
          ctx.restore();
        }
      }
    }

    // Animation loop
    function loop() {
      time += 0.031;
      drawKaleidoSpirals(time);
      requestAnimationFrame(loop);
    }

    // Initial paint
    loop();
</script>
