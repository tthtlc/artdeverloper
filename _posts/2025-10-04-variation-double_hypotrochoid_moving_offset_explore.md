---
layout: fullscreen
title: Pulsating Psychedelic Lissajous Webs
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
        margin: 0;
        background: radial-gradient(ellipse at center, #222 0%, #000 100%);
    }
    canvas {
        border: 2px solid #fff;
        margin-bottom: 18px;
        box-shadow: 0 0 30px #66fffc55;
        background: none;
    }
    #freqSliderA, #freqSliderB {
        width: 240px;
        margin-bottom: 8px;
        margin-right: 8px;
    }
    label {
        color: #ccffee;
        font-family: monospace;
        margin-right: 22px;
    }
    #startStopButton {
        padding: 10px 30px;
        font-size: 18px;
        font-family: inherit;
        border-radius: 7px;
        background: linear-gradient(90deg, #44c3f7 0%, #ff46cc 100%);
        color: #fff;
        border: none;
        box-shadow: 0 3px 12px #0ff5;
        cursor: pointer;
        margin-top: 10px;
        outline: none;
        transition: transform 0.15s;
    }
    #startStopButton:active {
        transform: scale(0.96);
    }
</style>

<canvas id="canvas" width="800" height="800"></canvas>
<div>
    <input type="range" id="freqSliderA" min="1" max="12" step="1" value="4">
    <label for="freqSliderA">Freq A</label>
    <input type="range" id="freqSliderB" min="1" max="12" step="1" value="6">
    <label for="freqSliderB">Freq B</label>
</div>
<button id="startStopButton">Start</button>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const freqSliderA = document.getElementById('freqSliderA');
const freqSliderB = document.getElementById('freqSliderB');
const startStopButton = document.getElementById('startStopButton');

let freqA = parseInt(freqSliderA.value);
let freqB = parseInt(freqSliderB.value);
freqSliderA.oninput = () => { freqA = parseInt(freqSliderA.value); };
freqSliderB.oninput = () => { freqB = parseInt(freqSliderB.value); };

let isAnimating = false;
let globalTime = 0;

const w = canvas.width;
const h = canvas.height;
const cx = w / 2, cy = h / 2;

function lissajous(t, a, b, delta, scale, amp) {
    // Lissajous curve parametric equation
    const x = scale * Math.sin(a * t + delta);
    const y = scale * Math.sin(b * t);
    return {x: cx + amp * x, y: cy + amp * y};
}

// Psychedelic color map (rainbow ripple)
function getColor(theta, t) {
    const p = (theta/Math.PI/2 + t/8) % 1;
    const r = Math.floor(180 + 76 * Math.sin(2 * Math.PI * p + t));
    const g = Math.floor(180 + 76 * Math.sin(2 * Math.PI * p + t + 2.1));
    const b = Math.floor(180 + 76 * Math.sin(2 * Math.PI * p + t + 4.2));
    return `rgba(${r},${g},${b},0.68)`;
}

// Draw evolving Lissajous "webs"
function drawWeb(time) {
    ctx.globalCompositeOperation = 'lighter';
    ctx.clearRect(0, 0, w, h);

    // Pulsing amplitude and inner modulation
    const ampBase = Math.min(w, h) * 0.36;
    const ampPulse = ampBase * (1 + 0.27 * Math.sin(time * 1.22));

    const waveLayers = 4;
    const webLines = 260;

    for (let layer = 0; layer < waveLayers; layer++) {
        const layerPhase = time*0.6 + layer * Math.PI / waveLayers;
        const r = 0.7 + 0.3*Math.sin(layerPhase + 2.3); // slightly different radii

        ctx.save();
        ctx.lineWidth = (2 + 1 * Math.cos(layerPhase + time/6.5));
        ctx.globalAlpha = 0.64 - 0.1 * layer;

        for (let i = 0; i < webLines; i++) {
            const t1 = 2*Math.PI * (i / webLines);
            const t2 = 2*Math.PI * ((i+1) / webLines);

            // The phase difference pulsates
            const delta = 0.72 * Math.PI * Math.sin(time*0.3 + layer*1.2);

            // Points on Lissajous
            const p1 = lissajous(t1, freqA+layer, freqB, delta+0.1*layer, r, ampPulse);
            const p2 = lissajous(t2, freqA+layer, freqB, delta+0.1*layer, r, ampPulse);

            ctx.beginPath();
            ctx.moveTo(p1.x, p1.y);
            ctx.lineTo(p2.x, p2.y);

            ctx.strokeStyle = getColor(t1, time + layer*0.98);
            ctx.shadowBlur = 16 + 4 * Math.sin(time+layer*1.5);
            ctx.shadowColor = ctx.strokeStyle;
            ctx.stroke();

            // Optionally, connect main points radially for a web look
            if (i % 16 === 0) {
                const q = lissajous(0, freqA+layer, freqB, delta+0.1*layer, r, ampPulse);
                ctx.beginPath();
                ctx.moveTo(cx, cy);
                ctx.lineTo(p1.x, p1.y);
                ctx.strokeStyle = getColor(t1, time - layer);
                ctx.lineWidth = 0.8;
                ctx.globalAlpha = 0.36;
                ctx.stroke();
            }
        }
        ctx.restore();
    }
}

function animate() {
    if (!isAnimating) return;
    globalTime += 0.017;
    drawWeb(globalTime);

    requestAnimationFrame(animate);
}

startStopButton.addEventListener('click', () => {
    isAnimating = !isAnimating;
    startStopButton.textContent = isAnimating ? 'Stop' : 'Start';
    if (isAnimating) animate();
});

// Draw initial frame
drawWeb(globalTime);
</script>
