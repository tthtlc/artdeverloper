---
layout: fullscreen
title: Psychedelic Hypnotic Vortex with Morphing Colorful Spirals
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Psychedelic Hypnotic Vortex with Morphing Colorful Spirals</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
    }
    canvas {
      border: 1px solid black;
    }
    .controls {
      display: flex;
      align-items: center;
      margin-top: 10px;
    }
    .label {
      margin-right: 10px;
      font-weight: bold;
    }
    .slider-value, .dropdown {
      margin-left: 10px;
    }
  </style>
</head>
<body>
  <canvas id="vortexCanvas" width="700" height="700"></canvas>
  <!-- Controls -->
  <div class="controls">
    <span class="label">Spirals:</span>
    <input type="range" id="spiralControl" min="2" max="16" value="7">
    <span id="spiralValue" class="slider-value">7</span>
    <div class="dropdown">
      <label class="label">Palette:</label>
      <select id="paletteSelector">
        <option value="acid">Acid</option>
        <option value="rainbow">Rainbow</option>
        <option value="icefire">Ice & Fire</option>
      </select>
    </div>
    <div class="dropdown">
      <label class="label">Radial Blur:</label>
      <input type="checkbox" id="blurToggle" checked>
    </div>
    <div class="dropdown">
      <label class="label">Center Color:</label>
      <input type="color" id="centerColorPicker" value="#000000">
    </div>
  </div>

  <script>
    const canvas = document.getElementById('vortexCanvas');
    const ctx = canvas.getContext('2d');
    const W = canvas.width;
    const H = canvas.height;
    const cx = W / 2, cy = H / 2;

    // Spiral/arms settings
    let spiralCount = 7;
    let baseSpiralCount = 7;
    let spiralPhase = 0;

    // Blur toggle
    let radialBlur = true;

    // Center color
    let centerColor = "#000000";

    // Palettes
    const palettes = {
      acid: [
        "#39ff14", "#ff007f", "#13e6ff", "#ffe700",
        "#ff00ff", "#00d0ff", "#30ffb7", "#e900ff"
      ],
      rainbow: [
        "#ff0000", "#ffa500", "#ffff00", "#00ff00",
        "#00ffff", "#0000ff", "#8b00ff", "#ff1493", "#ffdd00"
      ],
      icefire: [
        "#0ff0fc", "#31cdff", "#0058ff", "#291a6f",
        "#ff6a00", "#ff2300", "#ffcf00", "#ffe680"
      ]
    };
    let paletteKey = "acid";
    let palette = palettes[paletteKey];

    // Helper to interpolate between two hex colors
    function lerpColor(a, b, t) {
      const ah = +('0x' + a.slice(1)), bh = +('0x' + b.slice(1));
      const ar = (ah >> 16) & 0xff, ag = (ah >> 8) & 0xff, ab = ah & 0xff;
      const br = (bh >> 16) & 0xff, bg = (bh >> 8) & 0xff, bb = bh & 0xff;
      return `rgb(${
        Math.round(ar + (br - ar) * t)
      },${
        Math.round(ag + (bg - ag) * t)
      },${
        Math.round(ab + (bb - ab) * t)
      })`;
    }

    // Helper to blend current frame for fade-with-trail effect
    function fadeCanvas(alpha) {
      ctx.save();
      ctx.globalAlpha = alpha;
      ctx.globalCompositeOperation = "lighter";
      ctx.fillStyle = "#000001";
      ctx.fillRect(0, 0, W, H);
      ctx.restore();
    }

    // Morphing palette function (cycles hues)
    function shiftPalette(pal, t) {
      return pal.map(hex => {
        let c = +('0x' + hex.slice(1));
        let r = (c >> 16) & 0xff, g = (c >> 8) & 0xff, b = c & 0xff;
        // Shift hue using HSL
        let h, s, l;
        r /= 255, g /= 255, b /= 255;
        let max = Math.max(r,g,b), min = Math.min(r,g,b);
        l = (max+min)/2;
        if(max==min){
          h = s = 0;
        } else {
          let d = max-min;
          s = l > 0.5 ? d/(2-max-min) : d/(max+min);
          switch(max){
            case r: h = (g-b)/d+(g<b?6:0); break;
            case g: h = (b-r)/d+2; break;
            case b: h = (r-g)/d+4; break;
          }
          h /= 6;
        }
        // Morph hue
        h = (h + t) % 1;
        // Convert back to RGB
        let q = l < 0.5 ? l*(1+s) : l+s-l*s;
        let p = 2*l-q;
        function f(n){
          let k=n+1/3;
          if(k<0)k+=1; if(k>1)k-=1;
          if(k<1/6) return p+ (q-p)*6*k;
          if(k<1/2) return q;
          if(k<2/3) return p + (q-p)*(2/3 - k)*6;
          return p;
        }
        r = Math.round(255 * f(h));
        g = Math.round(255 * f(h-1/3));
        b = Math.round(255 * f(h-2/3));
        return `rgb(${r},${g},${b})`;
      });
    }

    // Render vortex
    function drawVortex(time) {
      // Quickly fade trails for a 'smearing' effect
      if(radialBlur) fadeCanvas(0.14);
      else ctx.clearRect(0, 0, W, H);

      // Morph palette
      let t = (time * 0.06) % 1;
      let pal = shiftPalette(palette, t);

      let arms = spiralCount;
      let nLines = 1600;
      let maxR = Math.min(W,H) * 0.46;

      // Spiraling point positions and their morphing radius/angle
      for(let j=0;j<arms;j++) {
        let armPhase = spiralPhase + (j/arms)*Math.PI*2 +
          Math.sin(time*0.004 + j) * 0.4
        ;
        let colorTop = pal[j % pal.length];
        let colorBottom = pal[(j+1)%pal.length];
        for(let i=0; i<nLines; i++) {
          let tR = i / nLines;
          let angle = armPhase +
            tR * Math.PI * 7 +
            Math.cos(Math.sin(time*0.001 + i/220 + j))*0.5 +
            Math.sin(time*0.0016 + (i/66) + j) * 0.15
          ;
          // Radial morphing for wave-vortex effect
          let r =
            maxR * tR
            * (0.62 + 0.12*Math.cos(i/14 + time*0.0013 + j))
            * (1 + 0.13*Math.sin(j + time*0.0021 + i/371))
            + 8 * Math.sin(i/5 + time*0.087)
          ;

          //  Hypnotic center waviness
          if (i < 80) {
            r *= 0.6 + 0.22*Math.cos(time*0.008 + i*0.13 + j*0.4);
          }

          // Animate slight twist
          angle += Math.sin(i/49 + time*0.003 + j*0.32) * 0.11;

          // Final position
          let x = cx + r * Math.cos(angle);
          let y = cy + r * Math.sin(angle);

          let blend = Math.pow(Math.sin(Math.PI*tR), 2.2); // so edges are sharper

          let clr = lerpColor(colorTop, colorBottom, blend);

          ctx.beginPath();
          ctx.arc(x, y, Math.max(0.6, 2-blend*1.5), 0, Math.PI*2);
          ctx.fillStyle = clr;
          ctx.globalAlpha = 0.9;
          ctx.fill();
        }
      }
      ctx.globalAlpha = 1.0;

      // Draw hypnotic morphing center
      let pulse = 24 + Math.sin(time*0.015) * 18 + Math.cos(time*0.03)*7;
      let centerRays = 42 + Math.floor(Math.abs(Math.sin(time*0.013))*17);
      for(let i=0; i<centerRays; i++) {
        let a = (i/centerRays)*Math.PI*2+Math.sin(time*0.01+i)*0.13;
        let r1 = 7 + Math.sin(time*0.08+i)*3;
        let r2 = pulse + Math.cos(time*0.032+i)*2;
        ctx.beginPath();
        ctx.moveTo(cx + r1*Math.cos(a), cy + r1*Math.sin(a));
        ctx.quadraticCurveTo(
          cx, cy,
          cx + r2*Math.cos(a), cy + r2*Math.sin(a)
        );
        ctx.strokeStyle = centerColor;
        ctx.lineWidth = (1.8 + Math.sin(i+time*0.18)*1.2);
        ctx.globalAlpha = 0.20 + 0.40 * Math.abs(Math.cos(i + time*0.07));
        ctx.stroke();
      }
      ctx.globalAlpha = 1.0;
      // Fill center
      ctx.beginPath();
      ctx.arc(cx, cy, pulse*0.62 + 2.5*Math.sin(time*0.026), 0, Math.PI*2);
      ctx.fillStyle = centerColor;
      ctx.globalAlpha = 0.17;
      ctx.fill();
      ctx.globalAlpha = 1.0;
    }

    // Animation loop
    let lastTime = 0;
    function animate(now) {
      let elapsed = now - lastTime;
      lastTime = now;
      let time = now || performance.now();

      spiralPhase += 0.003 * (spiralCount/6);

      // Subtle morph to spiral count -- breathing
      let morph = 0.28*Math.sin(time*0.00058);
      spiralCount = Math.max(2, Math.round(baseSpiralCount + morph));

      drawVortex(time);
      requestAnimationFrame(animate);
    }

    // Controls
    document.getElementById('spiralControl').addEventListener('input', (e)=>{
      baseSpiralCount = parseInt(e.target.value, 10);
      document.getElementById('spiralValue').textContent = baseSpiralCount;
    });
    document.getElementById('paletteSelector').addEventListener('change', (e)=>{
      paletteKey = e.target.value;
      palette = palettes[paletteKey];
    });
    document.getElementById('blurToggle').addEventListener('change', (e)=>{
      radialBlur = e.target.checked;
    });
    document.getElementById('centerColorPicker').addEventListener('input', (e)=>{
      centerColor = e.target.value;
    });

    // Initial center color
    centerColor = document.getElementById('centerColorPicker').value;

    // Start the animation!
    requestAnimationFrame(animate);
  </script>
</body>
</html>
```