---
layout: fullscreen
title: "Psychedelic Flower Waves"
tags:
  - graphics
---

<style>
    canvas {
        display: block;
        margin: 0 auto;
        background: radial-gradient(ellipse at center, #191921 0%, #232357 100%);
        /* fallback: */
        background-color: #191921;
        border-radius: 14px;
        margin-top: 24px;
        box-shadow: 0 4px 24px rgba(40,16,80,0.27);
    }
    .controls {
        display: flex;
        flex-direction: row;
        gap: 2rem;
        justify-content: center;
        align-items: center;
        margin-bottom: 16px;
        font-family: "Fira Mono", monospace;
        color: #b0aadd;
        font-size: 1.1em;
        text-shadow: 0 1px 4px #030f30;
    }
    label {
        margin-right: 8px;
    }
</style>
<div class="controls">
    <div>
        <label for="petalSlider">Petals:</label>
        <input id="petalSlider" type="range" min="3" max="24" value="7" step="1" oninput="updatePetals()">
        <span id="petalVal">7</span>
    </div>
    <div>
        <label for="waveSlider">Waves:</label>
        <input id="waveSlider" type="range" min="1" max="21" value="7" step="1" oninput="updateWaves()">
        <span id="waveVal">7</span>
    </div>
    <div>
        <label for="spreadSlider">Spread:</label>
        <input id="spreadSlider" type="range" min="8" max="240" value="88" step="1" oninput="updateSpread()">
        <span id="spreadVal">88</span>
    </div>
</div>
<canvas id="psyCanvas" width="600" height="600"></canvas>
<script>
let canvas = document.getElementById('psyCanvas');
let ctx = canvas.getContext('2d');
const petalSlider = document.getElementById('petalSlider');
const waveSlider = document.getElementById('waveSlider');
const spreadSlider = document.getElementById('spreadSlider');
const petalVal = document.getElementById('petalVal');
const waveVal = document.getElementById('waveVal');
const spreadVal = document.getElementById('spreadVal');

let W = canvas.width, H = canvas.height;
let t = 0;

let params = {
    petals: Number(petalSlider.value),
    waves: Number(waveSlider.value),
    spread: Number(spreadSlider.value)
};

function updatePetals() {
    params.petals = Number(petalSlider.value);
    petalVal.textContent = params.petals;
}
function updateWaves() {
    params.waves = Number(waveSlider.value);
    waveVal.textContent = params.waves;
}
function updateSpread() {
    params.spread = Number(spreadSlider.value);
    spreadVal.textContent = params.spread;
}
petalSlider.addEventListener('input', updatePetals);
waveSlider.addEventListener('input', updateWaves);
spreadSlider.addEventListener('input', updateSpread);

function draw() {
    t += 0.012;

    ctx.clearRect(0, 0, W, H);

    // Draw multiple overlapping flowerwave rings
    let rings = 6;
    for (let ring=0; ring<rings; ring++) {
        drawPetalWave(
            W/2, H/2,
            90 + ring * (params.spread/2),
            45 + ring * 17 + Math.sin(t+ring)*36,
            params.petals + ring,
            params.waves + ring,
            t*0.92 + ring*0.42,
            ring
        );
    }

    requestAnimationFrame(draw);
}

// Main flowerwave function
function drawPetalWave(cx, cy, baseRadius, maxOffset, petals, waves, time, ringNo) {
    const N = 180;
    ctx.save();
    ctx.translate(cx, cy);

    ctx.beginPath();
    for (let i=0; i<=N; i++) {
        let pct = i/N;
        let a = pct * Math.PI*2;

        // Dynamic petal & wave shapes
        let r =
            baseRadius
            + Math.sin(a*petals + time*1.5 + ringNo*1.1) * maxOffset * 0.34
            + Math.cos(a*waves + time*2.4 + 14*ringNo) * maxOffset * 0.63
            + Math.sin(time*0.7 + a*1.2 + ringNo) * 18 * Math.cos(pct*petals*1.2 + time);

        let x = r * Math.cos(a);
        let y = r * Math.sin(a);

        // Glow effect
        if(i===0)
            ctx.moveTo(x,y);
        else
            ctx.lineTo(x,y);
    }
    ctx.closePath();

    // Colorful HSL stroke/fill
    let hue = (
        200
        + 120 * Math.cos(time*0.53 + ringNo*1.4)
        + 60 * Math.sin(ringNo*2.4 + Math.sin(time+ringNo))
        + 48 * Math.sin(time*0.42 + ringNo*0.88)
    )%360;
    let sat = 80 + 10*Math.sin(time*0.6 + ringNo*0.7);
    let light = 55 + 10*Math.cos(time*0.8 + ringNo*0.52);

    ctx.shadowColor = `hsl(${(hue+62)%360},99%,85%)`;
    ctx.shadowBlur = 24 + 6*Math.sin(time+ringNo);

    ctx.globalAlpha = 0.58 + 0.36 * Math.cos(time*0.7 + ringNo);

    ctx.strokeStyle = `hsl(${(hue+160)%360}, 99%, 60%)`;
    ctx.lineWidth = 2.2 + ringNo * 0.8;
    ctx.stroke();

    ctx.globalAlpha = 0.28 + 0.21*Math.sin(time + ringNo);
    ctx.fillStyle = `hsl(${hue},${sat}%,${light}%)`;
    ctx.fill();

    ctx.shadowBlur = 0;
    ctx.globalAlpha = 1.0;
    ctx.restore();
}

draw();

</script>
