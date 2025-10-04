---
layout: fullscreen
title: Chromatic Wave Net – Lissajous Interference Playground
tags:
  - graphics
---

Interfering Chromatic Lissajous Net — Animate the fusion of waveforms!
<canvas id="canvas" width="800" height="800" style="border-radius: 16px;"></canvas>
<input type="range" id="freqSliderX" min="1" max="11" value="5" step="1">
<label for="freqSliderX">X freq</label>
<input type="range" id="freqSliderY" min="1" max="11" value="4" step="1">
<label for="freqSliderY">Y freq</label>
<button id="controlButton">Start</button>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const width = canvas.width;
const height = canvas.height;

const freqSliderX = document.getElementById('freqSliderX');
const freqSliderY = document.getElementById('freqSliderY');
const controlButton = document.getElementById('controlButton');

let freqX = parseInt(freqSliderX.value);
let freqY = parseInt(freqSliderY.value);
let isAnimating = false;
let t = 0;
let animationFrameId;

freqSliderX.addEventListener('input', () => { freqX = parseInt(freqSliderX.value); });
freqSliderY.addEventListener('input', () => { freqY = parseInt(freqSliderY.value); });

// Generate the points for two Lissajous figures
function lissajousPoints(a, b, delta, samples, phaseShift=0) {
    const points = [];
    for (let i = 0; i < samples; i++) {
        const t = (i / samples) * 2 * Math.PI;
        // Range [-1, 1]
        const x = Math.sin(a * t + delta + phaseShift);
        const y = Math.sin(b * t + phaseShift);
        points.push([x, y]);
    }
    return points;
}

// Map [-1,1] domain to canvas coordinates
function mapToCanvas(x, y) {
    const margin = 60;
    return [
        width / 2 + x * (width / 2 - margin),
        height / 2 + y * (height / 2 - margin),
    ];
}

function draw() {
    ctx.clearRect(0, 0, width, height);

    const samples = 220;

    // Oscillate delta slowly with t
    const delta = Math.PI / 2 + Math.sin(t * 0.21) * Math.PI / 3;
    const phaseWobble = Math.cos(t * 0.15) * Math.PI / 3;

    // Lissajous #1
    const points1 = lissajousPoints(freqX, freqY, delta, samples, 0);
    // Lissajous #2 (modulated)
    const modulatedFreqX = freqX + Math.sin(t * 0.17) * 1.2;
    const modulatedFreqY = freqY + Math.cos(t * 0.23) * 1.2;
    const points2 = lissajousPoints(modulatedFreqX, modulatedFreqY, -delta, samples, phaseWobble);

    // Draw "interference net" with chromatic color evolution
    ctx.save();
    ctx.globalAlpha = 0.7;
    for (let i = 0; i < samples; i++) {
        // Connect corresponding points
        const [x1, y1] = mapToCanvas(points1[i][0], points1[i][1]);
        const [x2, y2] = mapToCanvas(points2[i][0], points2[i][1]);

        ctx.beginPath();
        ctx.moveTo(x1, y1);
        ctx.lineTo(x2, y2);

        // Chromatic gradient: cycle hue across the shape, pulsate with t
        const hue = (360 * (i / samples) + t * 20) % 360;
        const sat = 86 + 10 * Math.sin(i / 17 + t * 0.4);
        const lum = 40 + 24 * Math.sin(i / 13 - t * 0.13);

        ctx.strokeStyle = `hsl(${hue},${sat}%,${lum}%)`;
        ctx.lineWidth = 1 + 1.8 * Math.abs(Math.sin(t + i / 79));
        ctx.stroke();
    }
    ctx.restore();

    // Draw each Lissajous loop as an evolving colored halo
    function drawLissajous(points, colorBase, alpha) {
        ctx.save();
        ctx.globalAlpha = alpha;
        ctx.beginPath();
        const [x0, y0] = mapToCanvas(points[0][0], points[0][1]);
        ctx.moveTo(x0, y0);
        for (let i = 1; i < points.length; i++) {
            const [xi, yi] = mapToCanvas(points[i][0], points[i][1]);
            ctx.lineTo(xi, yi);
        }
        ctx.closePath();
        ctx.shadowColor = colorBase;
        ctx.shadowBlur = 19;
        ctx.strokeStyle = colorBase;
        ctx.lineWidth = 4;
        ctx.stroke();
        ctx.restore();
    }
    // Halo colors pulse and shift
    const color1 = `hsl(${(t*15)%360},93%,51%)`;
    const color2 = `hsl(${(t*15+140)%360},95%,54%)`;
    drawLissajous(points1, color1, 0.23);
    drawLissajous(points2, color2, 0.21);
}

function animate() {
    if (!isAnimating) return;
    draw();
    t += 0.011;
    animationFrameId = requestAnimationFrame(animate);
}

controlButton.addEventListener('click', () => {
    isAnimating = !isAnimating;
    controlButton.textContent = isAnimating ? "Stop" : "Start";
    if (isAnimating) animate();
    else cancelAnimationFrame(animationFrameId);
});

// Initialize and render one frame
draw();
</script>
