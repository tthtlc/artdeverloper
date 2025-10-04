---
layout: fullscreen
title: Pulsating Psychedelic Wave Grids
tags:
  - graphics
---

Pulsating Psychedelic Wave Grids  
<div class="canvas-container">
<canvas id="psyWaveGridCanvas" width="600" height="600"></canvas>
<div id="gitalk-container"></div>
</div>

<script>
const canvas = document.getElementById('psyWaveGridCanvas');
const ctx = canvas.getContext('2d');
const w = canvas.width;
const h = canvas.height;

const cols = 22;
const rows = 22;
const cellSize = w / cols;
const timeScale = 0.6;
const colorSeed = Math.random() * 1000;

function psychedelicColor(t, phase) {
    // HSL cycling
    const h = ((t * 90 + phase) % 360 + 360) % 360;
    const s = 85 + 10 * Math.sin(t + phase / 90);
    const l = 50 + 30 * Math.cos(t + phase / 30);
    return `hsl(${h},${s}%,${l}%)`;
}

function drawWaveGrid(time) {
    ctx.clearRect(0, 0, w, h);

    let t = time * timeScale;
    for (let i = 0; i < cols; i++) {
        for (let j = 0; j < rows; j++) {
            // Each grid cell center
            const cx = i * cellSize + cellSize/2;
            const cy = j * cellSize + cellSize/2;

            // Wave parameters
            const waveX = Math.sin(t*0.6 + i*0.3 + j*0.5 + Math.sin(j*0.2 + t*0.72));
            const waveY = Math.cos(t*1.15 + j*0.45 + i*0.6 + Math.cos(i*0.2 + t*0.48));
            // Use waves to produce deformations
            const dx = waveX * 10 + Math.sin(t*0.47 + i*0.43 + j*0.21)*8;
            const dy = waveY * 10 + Math.cos(t*0.52 + j*0.32 + i*0.31)*8;
            // For the shape, blend sine/cos and draw star/flower/circle morphs
            const rBase = (cellSize/2 - 4) * (0.80 + 0.18 * Math.cos(t*0.15 + i*0.25 + j*0.34));
            const spokeCount = 5 + Math.floor(3 * Math.abs(Math.sin(t*0.21 + i*0.1 + j*0.14)));

            ctx.save();
            ctx.translate(cx+dx, cy+dy);

            // Layered shapes for psychedelic effect
            for (let layer=2; layer >= 0; layer--) {
                const r = rBase * (1 - layer*0.33) + 2*layer;
                const phase = colorSeed + t*40 + (i+layer)*40 + (j-layer)*55 + layer*12;
                ctx.beginPath();
                // Draw morph between flower/star and circle
                for (let k=0; k<=spokeCount*2; k++) {
                    // Use Lissajous-like modulation for radius
                    const ang = Math.PI * k / spokeCount;
                    const morph = Math.sin(ang*spokeCount + t*0.7 + i*0.15 - layer*0.22) * 0.36 + 0.64;
                    const rMorph = r * (0.75 + 0.22 * morph * Math.sin(t*0.65 + k + layer));
                    const x = Math.cos(ang) * rMorph;
                    const y = Math.sin(ang) * rMorph;
                    if (k === 0) ctx.moveTo(x, y);
                    else ctx.lineTo(x, y);
                }
                ctx.closePath();

                ctx.strokeStyle = psychedelicColor(t + i*0.13 + j*0.11 + layer*0.24, phase);
                ctx.lineWidth = 1.15 + layer * 1.1;
                ctx.shadowColor = ctx.strokeStyle;
                ctx.shadowBlur = 4 + layer*3;
                ctx.globalAlpha = 0.47 + 0.16 * layer;
                ctx.stroke();
            }
            ctx.restore();
        }
    }
    // Subtle glowing background pulse
    ctx.globalAlpha = 0.05 + 0.02*Math.sin(time*0.25);
    ctx.fillStyle = `hsl(${(colorSeed + t*18)%360},92%,15%)`;
    ctx.fillRect(0,0,w,h);
    ctx.globalAlpha = 1.0;

    requestAnimationFrame(drawWaveGrid);
}

drawWaveGrid(0);

</script>
