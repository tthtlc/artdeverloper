---
layout: fullscreen
title: "Psychedelic Moiré Ripples"
tags:
  - graphics
---

<style>
canvas {
    background-color: #181818;
    display: block;
    margin: 0 auto;
    border: 2px solid #444;
}
.controls {
    margin: 16px 0 10px 0;
    font-family: Arial, sans-serif;
    color: #eee;
    background: #232342;
    padding: 10px 12px;
    border-radius: 8px;
    width: fit-content;
    box-shadow: 0 2px 8px #0005;
}
.controls label {
    margin-right: 20px;
}
</style>

<div class="controls">
    <label for="waveCountRange">Waves: <span id="waveCountValue">5</span></label>
    <input type="range" id="waveCountRange" min="2" max="12" step="1" value="5">
    <label for="interferenceRange">Interf. <span id="interferenceValue">17</span></label>
    <input type="range" id="interferenceRange" min="5" max="30" step="1" value="17">
    <label for="speedRange">Speed <span id="speedValue">1.5</span></label>
    <input type="range" id="speedRange" min="0.2" max="4" step="0.1" value="1.5">
</div>
<canvas id="canvas" width="700" height="700"></canvas>
<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

// Controls
const waveCountRange = document.getElementById('waveCountRange');
const waveCountValue = document.getElementById('waveCountValue');
const interferenceRange = document.getElementById('interferenceRange');
const interferenceValue = document.getElementById('interferenceValue');
const speedRange = document.getElementById('speedRange');
const speedValue = document.getElementById('speedValue');

let waveCount = parseInt(waveCountRange.value);
let interference = parseInt(interferenceRange.value);
let speed = parseFloat(speedRange.value);

waveCountRange.addEventListener('input', () => {
    waveCount = parseInt(waveCountRange.value);
    waveCountValue.textContent = waveCount;
});
interferenceRange.addEventListener('input', () => {
    interference = parseInt(interferenceRange.value);
    interferenceValue.textContent = interference;
});
speedRange.addEventListener('input', () => {
    speed = parseFloat(speedRange.value);
    speedValue.textContent = speed;
});

const W = canvas.width;
const H = canvas.height;
const centerX = W / 2;
const centerY = H / 2;
const baseR = Math.min(W, H) * 0.41;

function lerp(a, b, t) {
    return a + (b - a) * t;
}

function drawMoireRipples(time) {
    ctx.clearRect(0, 0, W, H);
    ctx.save();
    ctx.globalCompositeOperation = "lighter";

    // Parameters for depth and flow
    const rings = 15 + Math.floor(waveCount * 0.8);
    const steps = 200;
    for (let k = 0; k < rings; k++) {
        const tRing = k / rings;
        const localR = lerp(baseR * 0.18, baseR * 1.04, tRing);

        // Dynamic coloring
        const hue = (time * 35 + k * 16 + Math.sin(time + k) * 37) % 360;
        const sat = 75 + 25 * Math.sin(time * 0.22 + k);
        const light = 56 + 23 * Math.sin(time * 0.37 - k);
        ctx.strokeStyle = `hsl(${hue},${sat}%,${light}%)`;
        ctx.lineWidth = lerp(1.8, 0.2, tRing);

        ctx.beginPath();
        for (let j = 0; j <= steps; j++) {
            const t = j / steps;
            const theta = t * Math.PI * 2;

            // Main interference - modulate radius
            let m1 = Math.sin(
                waveCount * theta
                + Math.sin(time * 0.6 + k * 0.6821) * 2
                + (k + time * 0.43) * 0.49
            );
            let m2 = Math.cos(
                interference * theta
                + Math.sin(time * 0.25 + k) * 5 +
                Math.sin(time * 0.16 - k * 0.4) * 2
            );
            let ripple = 1 + 0.16 * m1 + 0.16 * m2 +
                         0.04 * Math.sin(theta * waveCount * 2 + time * 0.73);

            const pulse = 1.0 + 0.25 * Math.sin(time * speed + k * 0.32);
            const r = localR * ripple * pulse;

            // Slight drift of the whole pattern
            const shiftTheta = theta + 0.02 * k * Math.sin(time * 0.29 + k);
            const x = centerX + r * Math.cos(shiftTheta);
            const y = centerY + r * Math.sin(shiftTheta);

            if (j === 0) {
                ctx.moveTo(x, y);
            } else {
                ctx.lineTo(x, y);
            }
        }
        ctx.closePath();
        ctx.shadowColor = `hsl(${(hue+25)%360},100%,70%)`;
        ctx.shadowBlur = lerp(0, 10, 1-tRing);
        ctx.stroke();
    }
    ctx.restore();

    // Soft center glow
    const gradient = ctx.createRadialGradient(centerX, centerY, 0, centerX, centerY, baseR * 1.25);
    gradient.addColorStop(0, `rgba(255,255,255,0.10)`);
    gradient.addColorStop(0.6, `rgba(70,30,80,0.02)`);
    gradient.addColorStop(1, "rgba(20,10,30,0.0)");
    ctx.save();
    ctx.globalAlpha = 0.8;
    ctx.globalCompositeOperation = "lighter";
    ctx.beginPath();
    ctx.arc(centerX, centerY, baseR * 1.23, 0, Math.PI * 2);
    ctx.closePath();
    ctx.fillStyle = gradient;
    ctx.fill();
    ctx.restore();
}

// Animation loop
let lastFrame = 0;
function animate(now) {
    const t = now * 0.001 * speed;
    drawMoireRipples(t);

    requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
</script>
