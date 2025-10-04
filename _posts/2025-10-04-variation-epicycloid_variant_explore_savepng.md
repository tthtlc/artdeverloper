---
layout: fullscreen
title: Hypnotic Quantum Rosefield
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hypnotic Quantum Rosefield</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: radial-gradient(circle at 50% 42%, #21293a 64%, #182023 100%);
        }
        canvas {
            border: 2px solid #181424;
            background: rgba(32,32,40,0.95);
            box-shadow: 0 0 32px #3df8c2, 0 0 2px #193;
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
            color: #ade;
            font-family: monospace;
            letter-spacing: 1px;
        }
        input[type="range"] {
            width: 180px;
        }
        .value-label {
            margin-left: 10px;
            font-weight: bold;
            color: #80ffd3;
            font-family: monospace;
        }
        #paletteBar {
            margin: 8px 0;
            width: 260px;
            height: 30px;
            border-radius: 12px;
            border: 1px solid #444;
        }
        button {
            margin-top: 10px;
            padding: 7px 24px;
            border-radius: 5px;
            border: none;
            color: #081;
            background: #b6ffe7;
            font-family: monospace;
            font-size: 1.05em;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 1px 3px #ccc;
        }
        button:active {
            background: #88fedd;
            color: #222;
        }
    </style>
</head>
<body>
    <canvas id="canvas" width="640" height="640"></canvas>
    <div class="controls">
        <div class="control-group">
            <label for="layers">Number of Layers:</label>
            <input type="range" id="layers" min="2" max="10" value="5">
            <span id="layers-value" class="value-label">5</span>
        </div>
        <div class="control-group">
            <label for="petals">Petal Complexity:</label>
            <input type="range" id="petals" min="3" max="14" value="7">
            <span id="petals-value" class="value-label">7</span>
        </div>
        <div class="control-group">
            <label for="particles">Particles per Layer:</label>
            <input type="range" id="particles" min="12" max="90" value="36">
            <span id="particles-value" class="value-label">36</span>
        </div>
        <div class="control-group">
            <label for="quantum">Quantum Field:</label>
            <input type="range" id="quantum" min="0" max="100" value="37">
            <span id="quantum-value" class="value-label">37</span>
        </div>
        <div class="control-group">
            <label for="palette">Color Palette:</label>
            <select id="palette">
                <option value="psy">Psychedelia</option>
                <option value="ocean">Ocean Dream</option>
                <option value="nebula">Nebula Pink</option>
                <option value="sunburst">Sunburst</option>
                <option value="rainbow">Rainbow Rings</option>
            </select>
        </div>
        <canvas id="paletteBar"></canvas>
        <button id="saveButton">Save as PNG</button>
    </div>
    <script>
        // Util Functions
        function lerp(a, b, t) {
            return a + (b - a) * t;
        }
        function hsvToRgb(h, s, v) {
            let f = (n, k = (h / 60 + n) % 6) =>
                v - v * s * Math.max(Math.min(k, 4 - k, 1), 0);
            let r = Math.round(f(5) * 255), g = Math.round(f(3) * 255), b = Math.round(f(1) * 255);
            return { r, g, b };
        }
        function rgbStr(c) {
            return `rgb(${c.r},${c.g},${c.b})`;
        }

        // Palette Definitions (all as arrays of H,S,V)
        const palettes = {
            psy: [
                {h:320,s:0.8,v:1}, {h:200,s:0.8,v:1}, {h:100,s:0.74,v:0.97}, {h:32,s:1,v:1}, {h:285,s:0.4,v:1}
            ],
            ocean: [
                {h:179,s:0.66,v:0.98}, {h:195,s:0.58,v:1}, {h:212,s:0.36,v:0.92}, {h:260,s:0.20,v:0.98}
            ],
            nebula: [
                {h:330,s:0.8,v:1}, {h:295,s:0.7,v:1}, {h:260,s:0.55,v:0.89}, {h:180,s:0.18,v:0.76}
            ],
            sunburst: [
                {h:355,s:0.92,v:1}, {h:41,s:0.86,v:1}, {h:59,s:0.92,v:0.99}, {h:32,s:0.92,v:1}
            ],
            rainbow: [
                {h:0,s:1,v:1},{h:30,s:1,v:1},{h:60,s:1,v:1},{h:120,s:1,v:1},{h:180,s:1,v:1},{h:240,s:1,v:1},{h:300,s:1,v:1}
            ]
        };

        // UI Elements
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const paletteBar = document.getElementById('paletteBar');
        const paletteCtx = paletteBar.getContext('2d');
        let width = canvas.width, height = canvas.height, cx = width/2, cy = height/2;

        // Animation State
        let t = 0;
        let layers = 5, petals = 7, particles = 36, quantum = 37;
        let paletteName = "psy";

        // UI Logic
        function uiUpdate() {
          document.getElementById('layers-value').innerText = layers;
          document.getElementById('petals-value').innerText = petals;
          document.getElementById('particles-value').innerText = particles;
          document.getElementById('quantum-value').innerText = quantum;
        }

        document.getElementById('layers').addEventListener('input', function() {
            layers = parseInt(this.value);
            uiUpdate();
        });
        document.getElementById('petals').addEventListener('input', function() {
            petals = parseInt(this.value);
            uiUpdate();
        });
        document.getElementById('particles').addEventListener('input', function() {
            particles = parseInt(this.value);
            uiUpdate();
        });
        document.getElementById('quantum').addEventListener('input', function() {
            quantum = parseInt(this.value);
            uiUpdate();
        });
        document.getElementById('palette').addEventListener('input', function() {
            paletteName = this.value;
            drawPaletteBar();
        });

        function drawPaletteBar() {
            const pal = palettes[paletteName];
            // Generate linear palette
            let grad = paletteCtx.createLinearGradient(0,0,paletteBar.width,0);
            for (let i=0; i<pal.length; ++i) {
                const rgb = hsvToRgb(pal[i].h, pal[i].s, pal[i].v);
                grad.addColorStop(i/(pal.length-1), rgbStr(rgb));
            }
            paletteCtx.fillStyle = grad;
            paletteCtx.clearRect(0,0,paletteBar.width,paletteBar.height);
            paletteCtx.fillRect(0,0,paletteBar.width,paletteBar.height);
        }
        drawPaletteBar();
        uiUpdate();

        // Quantum Rosefield Engine
        function drawQuantumRosefield() {
            ctx.clearRect(0,0,width,height);
            ctx.save();
            // Layer parameters
            const pal = palettes[paletteName];
            for (let L=0; L<layers; ++L) {
                // Vary radius with oscillation to make fields pulse and breathe
                const baseRad = lerp(120, 285, L/(layers-1));
                const waveRad = baseRad + 18 * Math.sin(t*0.35 + L * 0.8 + Math.sin(t/2)*0.6);
                // Petals phase modulation
                const petalPhase = t*0.25 + L*0.41 + Math.sin(t * 0.09 + L*0.7) * 0.2;
                let palMix = L/(layers-1);
                // Select color as smooth blend
                let pc0 = Math.floor(palMix * (pal.length-1)), pc1 = Math.min(pc0+1, pal.length-1);
                let frac = palMix*(pal.length-1)-pc0;
                let c0 = pal[pc0], c1 = pal[pc1];
                // Temporal color shift for added hallucination
                let delta = Math.sin(t*0.16 + L)*0.45 + 0.5;
                // Blend HSV
                const pick = {
                    h: lerp(c0.h,c1.h,frac)*delta + lerp(c1.h,c0.h,1-delta)*(1-delta),
                    s: lerp(c0.s, c1.s, frac),
                    v: lerp(c0.v, c1.v, frac)
                }
                const rgbStroke = hsvToRgb(pick.h, pick.s, pick.v);

                // Each quantum layer trail
                ctx.save();
                ctx.globalCompositeOperation = "lighter";
                ctx.globalAlpha = 0.14 + 0.40 * Math.sin(t*0.10 + L*0.55 + vfract(Math.sin(t*0.12 + L*0.29)));
                ctx.lineWidth = 3.1 - 2*L/(layers-1);
                ctx.strokeStyle = rgbStr(rgbStroke);
                ctx.beginPath();
                for (let p=0; p<=particles; p++) {
                    // Rose pattern (r = rad * sin(N*theta + phase))
                    let angle = (Math.PI * 2) * (p/particles);
                    let N = lerp(petals, petals*1.44 + L*0.5, Math.abs(Math.sin(t/7 + L)));
                    let M = quantum*0.01 + L*0.13 + 0.7 * Math.cos(t*0.15 + L*2.12);
                    let R = waveRad * Math.abs(Math.sin(N * angle + petalPhase));
                    // Quantum noise field
                    let qf = Math.sin(angle * M + Math.sin(t/3 + angle*0.45 + L*0.9) * (1.19+L*0.21));
                    // Fractal modulation
                    R *= 1 + 0.26*qf + 0.13*Math.sin(t*0.29+L+p*2.41);
                    // Some rumbling
                    R += 12*Math.sin(t*0.05+angle*2 + L*0.3);

                    // Polar to cartesian
                    let x = cx + R * Math.cos(angle + Math.sin(t*0.021+L));
                    let y = cy + R * Math.sin(angle);
                    if (p===0) ctx.moveTo(x,y);
                    else ctx.lineTo(x,y);
                }
                ctx.closePath();
                ctx.shadowColor = rgbStr(rgbStroke);
                ctx.shadowBlur = 18+L*7;
                ctx.stroke();
                ctx.restore();

                // Draw anchored glowing orbs at petal tips (higher layers get more modulation)
                for (let p=0; p<particles; p+=Math.max(1,Math.floor(particles/18))) {
                    let angle = (Math.PI * 2) * (p/particles);
                    let R = waveRad * Math.abs(Math.sin(petals * angle + petalPhase));
                    R *= 1 + 0.27*Math.sin(angle*M+Math.sin(t/2+L*0.1));
                    let x = cx + R * Math.cos(angle+Math.sin(t*0.021+L));
                    let y = cy + R * Math.sin(angle);
                    ctx.save();
                    ctx.globalAlpha = 0.18 + 0.27*Math.abs(Math.sin(angle*2+t + L*1.1));
                    ctx.beginPath();
                    ctx.arc(x,y, lerp(11,4,L/(layers-1)), 0, 2*Math.PI);
                    ctx.closePath();
                    // Glow gradiant
                    let orbcol = hsvToRgb((pick.h+35+L*12)%360, pick.s, 1);
                    let gradient = ctx.createRadialGradient(x,y,2,x,y,lerp(10,3,L/(layers-1)));
                    gradient.addColorStop(0, rgbStr(orbcol));
                    gradient.addColorStop(1, 'rgba(0,0,0,0)');
                    ctx.fillStyle = gradient;
                    ctx.fill();
                    ctx.restore();
                }
            }
            ctx.restore();
        }

        // Fract function for swirling envelopes
        function vfract(x) { return x - Math.floor(x); }

        // Animation Loop
        function animate() {
            t += 0.0275;
            drawQuantumRosefield();
            requestAnimationFrame(animate);
        }
        animate();

        document.getElementById('saveButton').addEventListener('click', ()=>{
            let link = document.createElement('a');
            link.download = 'quantum-rosefield-l'+layers+'-p'+petals+'-q'+quantum+'.png';
            link.href = canvas.toDataURL();
            link.click();
        });

        // Prevent right click context menu
        document.addEventListener('contextmenu', e => e.preventDefault());
    </script>
</body>
</html>
```
