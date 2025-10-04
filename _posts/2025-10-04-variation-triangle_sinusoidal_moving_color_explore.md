---
layout: fullscreen
title: Pulsating Psychedelic Hexagonal Web with Sinuous Filaments
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Psychedelic Pulsating Hex Web</title>
  <style>
    body {
      margin: 0;
      background: #090017;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
    }
    canvas {
      box-shadow: 0 0 24px #000b,
                  0 0 0 2px #0008;
      background: #110031;
      border-radius: 14px;
      display: block;
      margin-top: 30px;
    }
    .controls {
      margin-top: 20px;
      display: flex;
      gap: 18px;
      justify-content: center;
    }
    .label {
      color: #fdfdfd;
      font-weight: bold;
      letter-spacing: 1px;
      margin-right: 7px;
      font-family: 'Fira Mono', monospace;
      font-size: 1.1em;
    }
    .slider-value {
      margin-left: 7px;
      color: #faf0ff;
      font-family: 'Fira Mono', monospace;
    }
  </style>
</head>
<body>
  <canvas id="hexCanvas" width="630" height="630"></canvas>
  <div class="controls">
    <div>
      <span class="label">Filament Count:</span>
      <input type="range" id="filamentRange" min="3" max="30" value="12">
      <span class="slider-value" id="filamentVal">12</span>
    </div>
    <div>
      <span class="label">Motion:</span>
      <input type="range" id="motionRange" min="2" max="20" value="7">
      <span class="slider-value" id="motionVal">7</span>
    </div>
    <div>
      <span class="label">Palette:</span>
      <select id="paletteSelector">
        <option value="prism">Prism</option>
        <option value="acid">Acid</option>
        <option value="sublime">Sublime</option>
        <option value="cyber">Cyber</option>
      </select>
    </div>
  </div>
  <script>
    /*** SETUP ***/
    const canvas = document.getElementById('hexCanvas');
    const ctx = canvas.getContext('2d');
    let width = canvas.width;
    let height = canvas.height;
    const cx = width / 2, cy = height / 2;

    // Parameters
    let N_filaments = 12;
    let motionSpeed = 7;
    let fil_param = 0;

    // Control Elements
    const filRange = document.getElementById('filamentRange');
    const filVal = document.getElementById('filamentVal');
    filRange.addEventListener('input', e=>{
      N_filaments = +e.target.value;
      filVal.textContent = N_filaments;
    });

    const motionRange = document.getElementById('motionRange');
    const motionVal = document.getElementById('motionVal');
    motionRange.addEventListener('input', e=>{
      motionSpeed = +e.target.value;
      motionVal.textContent = motionSpeed;
    });

    /*** COLOR PALETTES (cyclic) ***/
    const palettes = {
      prism: [ "#FF004D", "#FF77AB", "#1BE7FF", "#6DFFE6", "#F7FF56", "#FFB20F" ],
      acid: [ "#09FF00", "#00FFD0", "#00C0FF", "#6100FF", "#F000FF", "#FF005B", "#FFDD00" ],
      sublime: [ "#4A90E2", "#50E3C2", "#B8E986", "#E0FF4F", "#FFA37A", "#F45E9F", "#804CF6" ],
      cyber: [ "#0FFFC0", "#0420FC", "#FF00FE", "#222C46", "#F9E900", "#53FF00" ]
    };
    let palette = palettes.prism;
    const paletteSel = document.getElementById('paletteSelector');
    paletteSel.addEventListener('change', e=>{
      palette = palettes[e.target.value];
    });

    // Hexagon Geometry
    const HEX_RADIUS = width * 0.37;
    // Place 6 vertices of hexagon
    function hexVertex(i) {
      const angle = Math.PI/3 * i - Math.PI/6; // rotate to "flat base"
      return {
        x: cx + HEX_RADIUS * Math.cos(angle),
        y: cy + HEX_RADIUS * Math.sin(angle)
      };
    }
    const vertices = [...Array(6)].map((_,i)=>hexVertex(i));

    /*** UTILS ***/
    function lerp(a, b, t) { return a + (b-a)*t; }
    // Cyclic Interp between palette (t in [0,1])
    function paletteColor(pal, t, a=1) {
      t = (t % 1 + 1) % 1; // wrap, ensure 0<=t<1
      const idx = t * pal.length;
      const i = Math.floor(idx);
      const j = (i+1)%pal.length;
      const frac = idx - i;
      // convert to rgb
      const c1 = pal[i], c2 = pal[j];
      function hex2rgb(hex) {
        hex = hex.replace('#','');
        return [parseInt(hex.slice(0,2),16),parseInt(hex.slice(2,4),16),parseInt(hex.slice(4,6),16)];
      }
      const [r1,g1,b1]=hex2rgb(c1), [r2,g2,b2]=hex2rgb(c2);
      const r=lerp(r1,r2,frac)|0, g=lerp(g1,g2,frac)|0, b=lerp(b1,b2,frac)|0;
      return `rgba(${r},${g},${b},${a})`;
    }

    /*** FILAMENT / WEB DRAWING ***/
    // Each filament is a snaky spline from hex center to a hex perimeter point, wriggling
    function drawFilament(theta, time, col_t) {
      const L = HEX_RADIUS*0.94;
      const steps = 130;

      ctx.beginPath();
      for (let i=0; i<=steps; ++i) {
        const st = i/steps;
        // Base radius from center to perimeter
        let r = st * L;
        // Each filament undulates in+out, with its own phase and frequency
        let fbase = 3 + 2*Math.sin(theta*6 + time*0.38);
        let amp = 18 + 6*Math.sin(theta*3 + time*0.11);
        let freq = fbase + Math.sin(time*0.037 + st*Math.PI*2)*1.4;
        let phase = time*0.15 + theta*5 + st*13.3;

        r += amp * Math.sin(freq*st + phase) * (1-st)*0.9;
        // circular warp to get "breathing"
        r *= 0.95 + 0.04*Math.sin(time*0.4 + Math.sin(theta)-st*2 + st*st*2);
        let x = cx + r * Math.cos(theta);
        let y = cy + r * Math.sin(theta);
        if (i===0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
      }
      ctx.strokeStyle = paletteColor(palette, col_t, 0.82 + 0.13*Math.sin(time*0.2+col_t*9));
      ctx.shadowColor = paletteColor(palette, col_t, 0.20);
      ctx.shadowBlur = 12 + 14*Math.abs(Math.sin(col_t*10+time*0.22));
      ctx.lineWidth = 3.2 + 2.2 * Math.abs(Math.sin(time*0.07 + col_t*23));
      ctx.stroke();
      ctx.shadowBlur = 0;
    }

    // Draw polygonal web with shifting thickness + color
    function drawWeb(time) {
      // for "pulsation", modulate polygonal "radii"
      let pulses = [];
      for (let i=0; i<6; ++i) {
        let pulse = 1 + 0.09 * Math.sin(time*0.22 + i*1.7 + Math.sin(time*0.1+i));
        pulses.push(pulse);
      }
      // Draw multiple web rings
      for (let layer=0; layer<4; ++layer) {
        let ringR = HEX_RADIUS * (0.42 + layer*0.18) * (0.95 + 0.03*Math.sin(time*0.24 + layer));
        ctx.beginPath();
        for (let i=0; i<=6; ++i) {
          let t = i/6;
          let theta = Math.PI/3 * i - Math.PI/6;
          let pulse = lerp(pulses[0],pulses[5],t) * (1-0.06*layer); // smooth
          let r = ringR * pulse;
          let x = cx + r * Math.cos(theta);
          let y = cy + r * Math.sin(theta);
          if (i===0) ctx.moveTo(x,y);
          else ctx.lineTo(x,y);
        }
        ctx.strokeStyle = paletteColor(palette, (time*0.037+layer*0.32)%1, 0.2+0.12*layer);
        ctx.lineWidth = 2.5 + 0.6*layer + 1.8*Math.abs(Math.sin(layer+time*0.38));
        ctx.shadowColor = paletteColor(palette,(layer*0.15+0.7)%1,0.12+0.18*Math.abs(Math.sin(time)));
        ctx.shadowBlur = 14-layer*3;
        ctx.stroke();
        ctx.shadowBlur = 0;
      }
    }

    /*** ANIMATION LOOP ***/
    function draw() {
      // Clear
      ctx.clearRect(0,0,width,height);

      // Subtle central glow
      let grad = ctx.createRadialGradient(cx,cy,0,cx,cy,width*0.51);
      grad.addColorStop(0,"#201448");
      grad.addColorStop(0.45,paletteColor(palette, (fil_param*0.013)%1,0.22));
      grad.addColorStop(1,"#09101d");
      ctx.fillStyle = grad;
      ctx.fillRect(0,0,width,height);

      // Hex web
      drawWeb(fil_param);

      // Filaments
      for (let i=0; i<N_filaments; ++i) {
        let theta = Math.PI*2 * i / N_filaments;
        // Color cycle per filament over time
        let col_t = ((i/N_filaments) + fil_param*0.028 + Math.sin(i*1.23 + fil_param*0.015)*0.09)%1;
        drawFilament(theta, fil_param, col_t);
      }

      /* Central hexagonal nucleus */
      ctx.save();
      ctx.translate(cx,cy);
      let spin = fil_param*0.045;
      ctx.rotate(spin);
      ctx.beginPath();
      for (let i=0; i<=6; ++i) {
        let ang = Math.PI/3 * i - Math.PI/6;
        let rad = 46 + 10*Math.sin(fil_param*0.13 + i);
        let x = rad * Math.cos(ang), y = rad * Math.sin(ang);
        if (i==0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
      }
      ctx.closePath();
      ctx.lineWidth = 6.3+2.1*Math.sin(fil_param*0.27);
      ctx.strokeStyle = paletteColor(palette, (fil_param*0.112)%1,0.65);
      ctx.shadowColor = paletteColor(palette, (fil_param*0.11+0.3)%1,0.34);
      ctx.shadowBlur = 12 + 20*Math.abs(Math.sin(fil_param*0.14));
      ctx.stroke();
      ctx.shadowBlur = 0;
      ctx.restore();
    }

    function animate() {
      fil_param += motionSpeed*0.045;
      draw();
      requestAnimationFrame(animate);
    }

    // Responsive resize, keep center
    window.addEventListener('resize',()=>{
      const min = Math.min(window.innerWidth, window.innerHeight) * 0.92;
      canvas.width = canvas.height = Math.max(430, Math.floor(min));
      width = canvas.width;
      height = canvas.height;
    });

    // Kickstart everything
    filVal.textContent = filRange.value;
    motionVal.textContent = motionRange.value;
    // Initial draw
    draw();
    animate();
  </script>
</body>
</html>
```