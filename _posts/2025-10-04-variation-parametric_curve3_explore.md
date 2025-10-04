---
layout: fullscreen
title: Psychedelic Lissajous Particle Vortex
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Psychedelic Lissajous Particle Vortex</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: radial-gradient(circle at 60% 35%, #111 70%, #320029 100%);
        }
        canvas {
            background: transparent;
            box-shadow: 0 2px 16px #0008;
            border-radius: 14px;
        }
        .controls {
            position: absolute;
            top: 30px;
            right: 30px;
            background: rgba(18,13,22,0.96);
            color: #edeaff;
            border-radius: 16px;
            box-shadow: 0 2px 16px #0005;
            padding: 18px 22px 16px 22px;
            font-family: 'Fira Mono', monospace;
            z-index: 2;
            min-width: 210px;
        }
        .control-group {
            display: flex;
            flex-direction: column;
            margin-bottom: 15px;
            gap: 3px;
        }
        label {
            font-size: 1em;
        }
        input[type="range"] {
            width: 160px;
        }
        span {
            color: #ffad80;
            font-size: .95em;
            font-weight: bold;
            margin-top: -2px;
        }
    </style>
</head>
<body>
    <canvas id="canvas" width="800" height="800"></canvas>
    <div class="controls">
        <div class="control-group">
            <label for="lissA">Lissajous A:</label>
            <input type="range" id="lissA" min="1" max="10" step="0.01" value="3.00">
            <span id="value-lissA">3.00</span>
        </div>
        <div class="control-group">
            <label for="lissB">Lissajous B:</label>
            <input type="range" id="lissB" min="1" max="10" step="0.01" value="2.45">
            <span id="value-lissB">2.45</span>
        </div>
        <div class="control-group">
            <label for="rotation">Vortex Curl:</label>
            <input type="range" id="rotation" min="0" max="2" step="0.005" value="1.13">
            <span id="value-rotation">1.13</span>
        </div>
        <div class="control-group">
            <label for="count">Particle Count:</label>
            <input type="range" id="count" min="30" max="300" step="1" value="106">
            <span id="value-count">106</span>
        </div>
    </div>
    <script>
        // For smooth color
        function hsl(h, s, l, a) {
            return `hsla(${h},${s}%,${l}%,${a})`;
        }

        // DOM elements
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const W = canvas.width, H = canvas.height;
        // Center at (cx, cy)
        const cx = W/2, cy = H/2;

        // Controls
        const params = {
            lissA: parseFloat(document.getElementById('lissA').value),
            lissB: parseFloat(document.getElementById('lissB').value),
            rotation: parseFloat(document.getElementById('rotation').value),
            count: parseInt(document.getElementById('count').value)
        };
        function updateDisplays() {
            document.getElementById('value-lissA').textContent = params.lissA.toFixed(2);
            document.getElementById('value-lissB').textContent = params.lissB.toFixed(2);
            document.getElementById('value-rotation').textContent = params.rotation.toFixed(2);
            document.getElementById('value-count').textContent = params.count;
        }
        document.getElementById('lissA').addEventListener('input', e=>{ params.lissA = parseFloat(e.target.value); updateDisplays(); });
        document.getElementById('lissB').addEventListener('input', e=>{ params.lissB = parseFloat(e.target.value); updateDisplays(); });
        document.getElementById('rotation').addEventListener('input', e=>{ params.rotation = parseFloat(e.target.value); updateDisplays(); });
        document.getElementById('count').addEventListener('input', e=>{ params.count = parseInt(e.target.value); updateDisplays(); });
        updateDisplays();

        // For smooth live adjustment:
        let particles = [];
        function makeParticles() {
            particles = [];
            for(let i=0; i<params.count; ++i) {
                const phi = 2*Math.PI*i/params.count;
                particles.push({
                    phase: phi,
                    radius: 170+100*Math.sin(phi*3+Math.sin(phi*5)),
                    size: 12+6*Math.cos(phi*8),
                    hue: (360*i/params.count),
                    offset: phi // for animated phase offset
                });
            }
        }
        // Initial population
        makeParticles();

        // Repopulate particles if count changed:
        let prevCount = params.count;
        // Animation time
        let t = 0;

        // Main draw
        function draw() {
            // Auto-regen if count changed
            if(params.count !== prevCount) {
                makeParticles();
                prevCount = params.count;
            }

            ctx.clearRect(0, 0, W, H);

            // Faded trails for motion blur
            ctx.globalAlpha = 0.25;
            ctx.fillStyle = "#130016";
            ctx.fillRect(0,0,W,H);
            ctx.globalAlpha = 1;

            // Draw all particles
            for(let i=0; i<particles.length; ++i) {
                const p = particles[i];

                // Lissajous core position (all particles move together, but offset)
                // Parametrize individual motion
                const S = 1.17 + 0.27*Math.sin(0.91*t + p.offset*5);
                // Lissajous loop:
                const lA = params.lissA, lB = params.lissB;
                const loopR = p.radius * (0.8 + 0.15*Math.sin(t*0.51 + p.offset*2));
                const x = cx + loopR * Math.cos(lA * t + p.phase) * S;
                const y = cy + loopR * Math.sin(lB * t + p.phase) * S;

                // Vortex swirl: apply extra angular rotation to particle around center
                const theta = params.rotation * t + p.phase + Math.sin(p.phase*2.9 + t*0.25)*0.2;

                // Spiral-out effect (pulsing in/out)
                const spiral = 1.1 + 0.27*Math.sin(t*0.85 + p.offset*7);
                const px = (x-cx)*Math.cos(theta)*spiral - (y-cy)*Math.sin(theta)*spiral + cx;
                const py = (x-cx)*Math.sin(theta)*spiral + (y-cy)*Math.cos(theta)*spiral + cy;

                // Particle color
                const hue = (p.hue + t*22 + 90*Math.sin(t*0.14 + p.offset*3.6))%360;
                const sat = 70+30*Math.cos(t*0.3+p.offset*5.05);
                const light = 56+22*Math.cos(p.phase*5 + t*0.6);
                const alpha = 0.84 + 0.14*Math.sin(t + p.offset*2);

                // Draw each as a glowing orb
                ctx.save();
                ctx.globalAlpha = alpha;
                let gr = ctx.createRadialGradient(px,py,0.5, px,py,p.size*1.9);
                gr.addColorStop(0, hsl(hue, sat, 94, 0.88));
                gr.addColorStop(0.32, hsl(hue, sat, light, 0.28));
                gr.addColorStop(1, "rgba(0,0,0,0)");
                ctx.beginPath();
                ctx.arc(px,py, p.size*1.9, 0, 2*Math.PI);
                ctx.fillStyle = gr;
                ctx.fill();
                ctx.restore();

                // Central solid core for pop
                ctx.save();
                ctx.beginPath();
                ctx.arc(px,py,p.size*0.57,0,2*Math.PI);
                ctx.fillStyle = hsl(hue, 72, light+8, 0.95);
                ctx.shadowColor = hsl(hue, 94, 50, 0.3);
                ctx.shadowBlur = 10;
                ctx.fill();
                ctx.restore();
            }
        }

        // Animation loop
        function animate() {
            t += 0.013;
            draw();
            requestAnimationFrame(animate);
        }

        animate();

        // Optionally, re-create particles at parameter change smoothly:
        document.getElementById('lissA').addEventListener('input',makeParticles);
        document.getElementById('lissB').addEventListener('input',makeParticles);
        document.getElementById('count').addEventListener('input',makeParticles);

        // Prevent context menu
        document.addEventListener("contextmenu", (e)=>{ e.preventDefault(); });

        // Responsiveness (optional): Not included for strict 800x800
    </script>
</body>
</html>
```
