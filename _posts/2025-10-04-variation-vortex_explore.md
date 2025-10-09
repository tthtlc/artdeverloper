---
layout: fullscreen
title: "Psychedelic Sunflower Moiré Animation"
tags:
  - graphics
---

<style>
    body {
        background: radial-gradient(circle, #18082c 0%, #00061c 100%);
        margin: 0;
    }
    canvas {
        display: block;
        margin: 32px auto;
        background: transparent;
        border-radius: 16px;
        box-shadow: 0 0 32px rgba(80,30,150,0.45);
    }
    .controls {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 8px;
        margin-bottom: 8px;
    }
    .control-group {
        display: flex;
        flex-direction: column;
        align-items: center;
        color: #f0e9fe;
        font-family: 'Fira Mono', monospace;
        font-size: 1.1em;
    }
    label {
        letter-spacing: 1px;
        margin-bottom: 2px;
    }
    input[type="range"] {
        width: 200px;
    }
    #canvas {
        background: transparent;
        border: 1.5px solid #5d39f7;
    }
</style>

<div class="controls">
        <div class="control-group">
            <label for="petals">Petals:</label>
            <input type="range" id="petals" min="5" max="32" step="1" value="16">
            <span id="value-petals">16</span>
        </div>
        <div class="control-group">
            <label for="density">Density:</label>
            <input type="range" id="density" min="200" max="1800" step="10" value="960">
            <span id="value-density">960</span>
        </div>
        <div class="control-group">
            <label for="wave">Wave:</label>
            <input type="range" id="wave" min="0" max="100" step="1" value="33">
            <span id="value-wave">33</span>
        </div>
        <div class="control-group">
            <label for="colorize">Color:</label>
            <input type="checkbox" id="colorize" checked>
        </div>
</div>
<canvas id="canvas" width="800" height="800"></canvas>
<script>
        document.addEventListener("contextmenu", function(event) { event.preventDefault(); });

        // ======= Controls =======
        function updateValueDisplay(id, value) {
            document.getElementById(`value-${id}`).textContent = value;
        }
        document.querySelectorAll('input[type="range"]').forEach((slider) => {
            slider.addEventListener('input', () => {
                updateValueDisplay(slider.id, slider.value);
            });
        });

        // ======= Canvas Setup =======
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        const W = canvas.width, H = canvas.height;
        ctx.translate(W/2, H/2);

        // ======= Parameters =======
        let petals = Number(document.getElementById('petals').value);
        let density = Number(document.getElementById('density').value);
        let waveAmp = Number(document.getElementById('wave').value);
        let colorize = document.getElementById('colorize').checked;

        // Controls handling
        document.getElementById('petals').addEventListener('input', e=>{
            petals = Number(e.target.value);
        });
        document.getElementById('density').addEventListener('input', e=>{
            density = Number(e.target.value);
        });
        document.getElementById('wave').addEventListener('input', e=>{
            waveAmp = Number(e.target.value);
        });
        document.getElementById('colorize').addEventListener('input', e=>{
            colorize = !!e.target.checked;
        });

        // ======= Animation State =======
        let t0 = Date.now();
        function animate() {
            const now = Date.now();
            const elapsed = (now-t0)/1000;
            // Slight persistent fade for trails:
            ctx.globalAlpha = 0.21;
            ctx.fillStyle = "rgba(12,1,34,0.18)";
            ctx.fillRect(-W/2, -H/2, W, H);
            ctx.globalAlpha = 1.0;

            // Main Moiré Sunflower
            // Equations derived from phyllotaxis + modulated radius
            const φ = Math.PI * (3 - Math.sqrt(5)); // golden angle (≈ 137.5°)
            const K = 370; // scale radius

            for(let i=0;i<density;++i){
                let theta = i * φ;
                let r = Math.sqrt(i/density) * K;
                // Wobble on radius, frequency set by petals, modulated over time
                let osc = Math.sin( (petals + 1) * theta + Math.cos(elapsed/3) )
                          * (waveAmp/22)
                          * Math.sin(theta + Math.cos(elapsed/2));
                let x = (r + osc*18) * Math.cos(theta);
                let y = (r + osc*18) * Math.sin(theta);

                // Animate pulse to breathing effect:
                let scalePulse = 0.97 + 0.03 * Math.sin(elapsed * 2 + theta * petals);
                x *= scalePulse;
                y *= scalePulse;

                // Moiré: connect spiral to a mirrored counterpart
                let mirrorTheta = -theta + 0.48 * Math.sin(elapsed/1.3+theta);
                let mirrorR = r + osc*13;
                let mx = (mirrorR) * Math.cos(mirrorTheta);
                let my = (mirrorR) * Math.sin(mirrorTheta);

                // Colorful shifting
                if(colorize){
                    let hue = ((theta*petals/Math.PI + elapsed*25) % 360+360)%360;
                    let sat = 75 + 21 * Math.sin(elapsed + theta*1.7);
                    let light = 56 + 21 * Math.cos(theta - elapsed*0.9);
                    ctx.strokeStyle = `hsl(${hue},${sat}%,${light}%)`;
                } else {
                    let g = Math.floor(144+(i/density)*80);
                    ctx.strokeStyle = `rgb(${g},${g},${g})`;
                }

                ctx.beginPath();
                ctx.moveTo(x, y);
                ctx.lineTo(mx, my);
                ctx.stroke();

                // Draw glowing dot at endpoint
                if (i%9===0){
                    if(colorize){
                        let h = ((theta*petals/Math.PI*3 + elapsed*35) % 360+360)%360;
                        ctx.fillStyle = `hsl(${h},97%,55%)`;
                    } else {
                        ctx.fillStyle = "#fff9";
                    }
                    ctx.beginPath();
                    ctx.arc(x, y, 3+1.5*Math.abs(osc), 0, 2*Math.PI);
                    ctx.fill();
                }
            }

            // Draw overlay core
            ctx.save();
            ctx.globalAlpha = 0.77;
            let corePulse = 41 + 2*Math.sin(elapsed*1.8);
            if(colorize) ctx.fillStyle = 'hsl(' + ((elapsed*65)%360) + ',70%,60%)';
            else ctx.fillStyle = "#eee";
            ctx.beginPath();
            ctx.arc(0,0,corePulse,0,2*Math.PI);
            ctx.fill();
            ctx.restore();

            requestAnimationFrame(animate);
        }

        // ======= Initial Update =======
        updateValueDisplay('petals', petals);
        updateValueDisplay('density', density);
        updateValueDisplay('wave', waveAmp);

        // ======= Start =======
        requestAnimationFrame(animate);
</script>
