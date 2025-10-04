---
layout: fullscreen
title: "Flowing Psychedelic Moiré Spirals"
tags:
  - graphics
---

<style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      background: #101015;
      margin: 0;
    }
    canvas {
      border: 2px solid #333;
      background: #181828;
      box-shadow: 0 0 20px #000a;
    }
    .controls {
      margin-top: 14px;
      display: flex;
      gap: 22px;
      align-items: center;
    }
    .label {
      margin-right: 7px;
      font-weight: 600;
      color: #ffeedd;
    }
    .slider-value, .dropdown {
      margin-left: 6px;
    }
</style>
<canvas id="spiralCanvas" width="720" height="720"></canvas>
<div class="controls">
    <span class="label">Spiral Count:</span>
    <input type="range" min="1" max="12" value="5" id="spiralCountRange">
    <span id="spiralCountValue" class="slider-value">5</span>

    <span class="label">Moire Wave:</span>
    <input type="range" min="1" max="10" value="4" id="moireRange">
    <span id="moireValue" class="slider-value">4</span>

    <div class="dropdown">
      <label class="label" for="paletteSelect">Palette:</label>
      <select id="paletteSelect">
        <option value="acidRave">Acid Rave</option>
        <option value="sunsetDream">Sunset Dream</option>
        <option value="nebula">Nebula</option>
        <option value="jungleTrip">Jungle Trip</option>
      </select>
    </div>
    <div class="dropdown">
      <label class="label" for="blendModeSelect">Blend:</label>
      <select id="blendModeSelect">
        <option value="lighter">Additive</option>
        <option value="multiply">Multiply</option>
        <option value="screen">Screen</option>
        <option value="difference">Difference</option>
      </select>
    </div>
</div>
<script>
    const canvas = document.getElementById('spiralCanvas');
    const ctx = canvas.getContext('2d');
    const w = canvas.width, h = canvas.height;
    const cx = w / 2, cy = h / 2;
    let spiralCount = 5;
    let moireWave = 4;
    let time = 0;

    // Color palettes
    const palettes = {
      acidRave: [
        "#fff600", "#fa8bff", "#2BD2FF", "#FF5E5B", "#C0FF00", "#00FFA1", "#FF61A6"
      ],
      sunsetDream: [
        "#ffb347", "#ffcc33", "#fd6a6a", "#ff7eb3", "#e33bff", "#6fd0ff"
      ],
      nebula: [
        "#3e206e", "#4831d4", "#ccf381", "#7fdbda", "#ff67e7", "#f6f6f6"
      ],
      jungleTrip: [
        "#006a71", "#219c90", "#cbf078", "#fec601", "#faa613"
      ]
    };

    let selectedPalette = palettes.acidRave;
    let blending = "lighter";

    // Moiré parameters
    let baseMoire = 4;

    // UI Controls
    document.getElementById('spiralCountRange').addEventListener('input', e => {
      spiralCount = Number(e.target.value);
      document.getElementById('spiralCountValue').textContent = spiralCount;
    });
    document.getElementById('moireRange').addEventListener('input', e => {
      moireWave = Number(e.target.value);
      document.getElementById('moireValue').textContent = moireWave;
    });
    document.getElementById('paletteSelect').addEventListener('change', e => {
      selectedPalette = palettes[e.target.value];
    });
    document.getElementById('blendModeSelect').addEventListener('change', e => {
      blending = e.target.value;
    });

    // Color interpolation helpers
    function hexToRgb(hex) {
      hex = hex.replace("#", "");
      if (hex.length === 3) hex = hex.split("").map(s=>s+s).join("");
      let num = parseInt(hex, 16);
      return [
        (num >> 16) & 255,
        (num >> 8)  & 255,
        (num)       & 255
      ];
    }
    function rgbToHex([r,g,b]) {
      r = ("0"+r.toString(16)).slice(-2);
      g = ("0"+g.toString(16)).slice(-2);
      b = ("0"+b.toString(16)).slice(-2);
      return "#" + r + g + b;
    }
    function lerp(a,b,t) {
      return a + (b-a)*t;
    }
    function lerpColor(a,b,t) {
      a = hexToRgb(a); b = hexToRgb(b);
      return rgbToHex([
        Math.round(lerp(a[0],b[0],t)),
        Math.round(lerp(a[1],b[1],t)),
        Math.round(lerp(a[2],b[2],t))
      ]);
    }
    function getPaletteColor(idx, palette, t) {
      // Cyclic gradient over the palette
      let seg = palette.length;
      let p = (idx + t*seg)%seg;
      let i = Math.floor(p) % seg;
      let j = (i+1)%seg;
      let fr = p-i;
      return lerpColor(palette[i], palette[j], fr);
    }

    // Main spiral drawing logic
    function drawSpiral(cx, cy, baseR, angleOffset, spiralIdx, moireFreq, t) {
      ctx.save();
      let arms = 1;
      let points = 1200;
      let spread = Math.PI*2*arms;
      for (let pt = 0; pt < points; pt++) {
        let theta = spread * (pt/points) + angleOffset;
        // The moiré + sine-warped radius
        let r = baseR + 
          Math.sin(theta*spiralCount + t*1.7 + spiralIdx*1.31) * 62 +
          Math.cos((theta*moireFreq + t*1.8 + spiralIdx*0.8)) * 90 * Math.sin(t*0.24+spiralIdx);
        // Extra radial vibration for more psychedelia
        r += Math.sin(pt*0.017 + theta*1.7 + t*2.12 + spiralIdx*3)*(18+Math.sin(spread*pt/points+t)*6);
        // Color cycling along the arm
        let colT = (pt/points + t*0.044 + spiralIdx*0.09) % 1;
        let color = getPaletteColor(spiralIdx, selectedPalette, colT);
        ctx.beginPath();
        ctx.arc(
          cx + r*Math.cos(theta),
          cy + r*Math.sin(theta),
          2.5 + Math.cos(theta*3 + t*1.5 + spiralIdx)*1.2,
          0, Math.PI*2
        );
        ctx.fillStyle = color;
        ctx.globalAlpha = 0.66+Math.sin(pt*0.009+t+spiralIdx)*0.18;
        ctx.fill();
      }
      ctx.restore();
    }

    // Dynamic & softly pulsating background
    function drawBackground(t) {
      // Soft vertical color sweep
      let grd = ctx.createLinearGradient(0, 0, w, h);
      let stops = 6;
      for(let i=0;i<stops;++i) {
        let p = i/(stops-1);
        let color = getPaletteColor(i, selectedPalette, (t*0.09+i*0.12) % 1);
        grd.addColorStop(p, color+"aa");
      }
      ctx.globalAlpha = 0.24+0.05*Math.cos(t*0.8);
      ctx.fillStyle = grd;
      ctx.fillRect(0,0,w,h);
      ctx.globalAlpha = 1.0;
    }

    // Animation loop
    function animate() {
      time += 0.0166; // ≈60FPS
      // Fade out previous frame slightly for trails
      ctx.globalCompositeOperation = "source-over";
      ctx.fillStyle = "#181828";
      ctx.globalAlpha = 0.28 + 0.04*Math.sin(time*0.7);
      ctx.fillRect(0, 0, w, h);
      ctx.globalAlpha = 1;

      drawBackground(time);

      ctx.globalCompositeOperation = blending;
      let mainRadius = 158 + 16*Math.sin(time*0.45);
      for (let s = 0; s < spiralCount; s++) {
        let aShift = (Math.PI*2 / spiralCount) * s + time*0.16 + Math.sin(s+time*0.42)*0.16;
        let moireF = moireWave+Math.sin(time*0.13+s)*0.5+s*0.11;
        drawSpiral(cx, cy, mainRadius, aShift, s, moireF, time);
      }
      ctx.globalCompositeOperation = "source-over";
      requestAnimationFrame(animate);
    }

    // UI: show starting values
    document.getElementById('spiralCountValue').textContent = spiralCount;
    document.getElementById('moireValue').textContent = moireWave;

    // Start!
    animate();
</script>
