```markdown
---
layout: fullscreen
title: "Psychedelic Hypnotic Spiral Waves"
tags:
  - graphics
---

<canvas id="spiralCanvas" width="800" height="800"></canvas>
<script>
const canvas = document.getElementById('spiralCanvas');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const W = canvas.width;
const H = canvas.height;

// --- PARAMETERS ---
let t = 0;
const layers = 7; // Number of spiral wave layers
const pointsPerSpiral = 260;
const spiralDensity = 1.5; // Higher = denser spirals
const spiralFreq = 3; // More freq = more winding
const baseRadius = Math.min(W, H) * 0.13;

function lerp(a,b,n) { return a + (b-a)*n; }

// --- RANDOM COLOR PALETTE GENERATOR ---
function hsl(vibrancy, offset = 0) {
    // vibrancy in [0, 1], offset in [0, 360]
    let hue = ((vibrancy * 320 + offset) % 360);
    return `hsl(${hue}, 80%, 55%)`;
}

// --- SPIRAL LAYER DRAWING ---
function drawSpiralLayer(centerX, centerY, radius, phase, color, thickness, frequency, density) {
    ctx.save();
    ctx.beginPath();
    for(let i = 0; i < pointsPerSpiral; i++) {
        let p = i / (pointsPerSpiral-1);
        let theta = density * 2 * Math.PI * p * frequency + phase;
        let wave = Math.sin(theta * 0.5 + phase) * (radius * 0.22) + Math.cos(theta * 0.87 + phase) * (radius * 0.18);
        let r = radius + wave;
        let x = centerX + r * Math.cos(theta);
        let y = centerY + r * Math.sin(theta);
        if(i === 0)
            ctx.moveTo(x, y);
        else
            ctx.lineTo(x, y);
    }
    ctx.strokeStyle = color;
    ctx.shadowColor = color;
    ctx.shadowBlur = 18 * thickness;
    ctx.lineWidth = thickness;
    ctx.globalAlpha = 0.8;
    ctx.stroke();
    ctx.restore();
}

function drawHypnoticSpiral(tick) {
    ctx.clearRect(0, 0, W, H);
    let cx = W / 2, cy = H / 2;
    let tAbs = tick * 0.002;
    // Background glow
    let grad = ctx.createRadialGradient(cx, cy, 0, cx, cy, W*0.6);
    grad.addColorStop(0, "#12041d");
    grad.addColorStop(1, "#333044");
    ctx.fillStyle = grad;
    ctx.fillRect(0,0,W,H);

    // Draw overlapping spiral-waves
    for(let k = 0; k < layers; k++) {
        let prog = k / (layers-1);
        let spiralPhase = tAbs * lerp(0.5, 1.3, prog) - prog * Math.PI * 2.3 + Math.sin(tAbs*0.8 + k*1.11) * 1.2;
        let color = hsl(prog, Math.sin(tAbs*0.6 + prog*6.1) * 110 + lerp(0, 190, prog));
        let thickness = lerp(9, 2, prog) + Math.sin(tAbs*0.8+prog*5)*2;
        let frequency = spiralFreq + Math.sin(tAbs*0.3 + k*0.41)*0.88;
        let density = spiralDensity + Math.sin(tAbs*0.21 + k*2.17)*0.5;
        let rad = baseRadius * lerp(0.9, 2.7, prog) + Math.sin(tAbs*0.38 + k*1.82) * 9;
        drawSpiralLayer(cx, cy, rad, spiralPhase, color, thickness, frequency, density);

        // Optionally, echo outline in highlight
        if (k % 2 === 0) {
            let glowColor = hsl(prog, 230 + Math.cos(tAbs*1.22+prog*2)*120);
            drawSpiralLayer(cx, cy, rad + 8, spiralPhase + 0.21, glowColor, 1.3, frequency + 0.4, density + 0.2);
        }
    }

    // Central eye vortex effect (pulsing circles and radiants)
    let pulse = Math.sin(tAbs*2)*0.5+0.5;
    let r0 = baseRadius * (0.57 + pulse*0.08);
    for(let i=0;i<3;i++) {
        ctx.save();
        ctx.beginPath();
        ctx.arc(cx, cy, r0*(1+i*0.34), 0, 2*Math.PI);
        ctx.lineWidth = (2.5-i*0.8)+(pulse*2);
        ctx.globalAlpha = 0.11+0.09*i;
        ctx.strokeStyle = hsl(0.17+(pulse*0.46)+i*0.17, 160-i*42);
        ctx.shadowColor = ctx.strokeStyle;
        ctx.shadowBlur = 20;
        ctx.stroke();
        ctx.restore();
    }

    // Draw radiating lines (pulsating "spokes")
    let numSpokes = 11;
    for (let i = 0; i < numSpokes; i++) {
        let angle = (i / numSpokes * Math.PI * 2) + Math.sin(tAbs*0.4 + i*0.63)*0.2;
        let len = baseRadius * (1.1+Math.sin(tAbs*1.2 + i*0.43)*0.18);
        ctx.save();
        ctx.beginPath();
        ctx.moveTo(cx, cy);
        ctx.lineTo(cx + Math.cos(angle) * len, cy + Math.sin(angle) * len);
        ctx.globalAlpha = 0.11 + 0.18* Math.abs(Math.sin(tAbs+i*0.5));
        ctx.strokeStyle = hsl(i/numSpokes, 30 + Math.abs(Math.cos(tAbs*0.8 + i)));
        ctx.lineWidth = 1.9 + Math.abs(Math.sin(tAbs*0.7 + i));
        ctx.shadowColor = ctx.strokeStyle;
        ctx.shadowBlur = 13;
        ctx.stroke();
        ctx.restore();
    }
}

function animate() {
    t++;
    drawHypnoticSpiral(t);
    requestAnimationFrame(animate);
}

animate();

// Make responsive
window.addEventListener('resize', () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
});
</script>
```
