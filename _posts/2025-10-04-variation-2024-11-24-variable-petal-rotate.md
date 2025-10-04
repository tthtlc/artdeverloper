---
layout: fullscreen
title: "Spiraling Wavefield Pulse Animation"
tags:
  - graphics
---

<style>
canvas {
    display: block;
    margin: 30px auto 0 auto;
    background: radial-gradient(ellipse at 50% 60%, #3e0066 0%, #000010 100%);
    box-shadow: 0 0 32px #130030b2;
    border-radius: 20px;
}
.controls {
    margin: 10px auto;
    font-family: Arial, sans-serif;
    text-align: center;
    color: #eee;
}
.controls label {
    margin-right: 10px;
}
</style>

<div class="controls">
    <label for="waveCount">Wave Count: <span id="waveValue">2</span></label>
    <input type="range" id="waveCount" min="1" max="8" step="1" value="2">
    <label for="speed">Speed: <span id="speedValue">1.0</span></label>
    <input type="range" id="speed" min="0.2" max="4" step="0.01" value="1.0">
</div>
<canvas id="animationCanvas" width="700" height="700"></canvas>
<script>
const canvas = document.getElementById('animationCanvas');
const ctx = canvas.getContext('2d');
const waveSlider = document.getElementById('waveCount');
const waveValue = document.getElementById('waveValue');
const speedSlider = document.getElementById('speed');
const speedValue = document.getElementById('speedValue');

let waveCount = +waveSlider.value;
let speed = +speedSlider.value;
waveSlider.addEventListener('input', ()=>{
    waveCount = +waveSlider.value;
    waveValue.textContent = waveCount;
});
speedSlider.addEventListener('input', ()=>{
    speed = +speedSlider.value;
    speedValue.textContent = (+speed).toFixed(2);
});

// Parameters and palette
const NUM_PTS = 300; // points per spiral
const NUM_RINGS = 5; // number of overlapping spirals
const MAX_RADIUS = 300;
const COLORS = [
    "#5e4fa2","#3288bd","#66c2a5","#abdda4","#e6f598",
    "#fee08b","#fdae61","#f46d43","#d53e4f","#9e0142"
];
let time = 0;

function drawSpiralingWavefield(ctx, t, ctrl) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    let cx = canvas.width/2, cy = canvas.height/2;
    ctx.save();
    ctx.globalCompositeOperation = 'lighter';

    for (let ring = 0; ring < NUM_RINGS; ++ring) {
        // Stagger time for multi-ring interference
        const tRing = t + ring * 0.75;
        // Slightly offset each spiral
        const spiralOffset = (Math.PI * 2 / NUM_RINGS) * ring;
        // Pulsing alpha
        let alpha = 0.16 + 0.14*Math.sin(t*1.5 + ring);

        ctx.beginPath();
        for (let i = 0; i < NUM_PTS; ++i) {
            // Spiral radius and angle
            const frac = i/NUM_PTS;
            let theta = 2 * Math.PI * ctrl.waveCount * frac + tRing + spiralOffset;
            let baseR = MAX_RADIUS * frac;

            // Radial waveform, dizzy sinus pulsation
            let wmod1 = Math.sin(4*theta + Math.sin(tRing*0.8+frac*6.3)*2);
            let wmod2 = Math.cos(ctrl.waveCount * theta*0.6 + tRing*0.6 - frac*4);
            let wmod = wmod1*0.22 + wmod2*0.22;

            let radius = baseR + Math.sin(theta + tRing*1.3)*28 + wmod*56
                + 32 * Math.sin(frac*8+0.8*tRing);

            let x = cx + Math.cos(theta) * radius;
            let y = cy + Math.sin(theta) * radius;

            if (i===0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        ctx.closePath();

        let cIdx = (ring*2 + Math.floor(Math.sin(tRing)*5) + COLORS.length)%COLORS.length;
        ctx.strokeStyle = COLORS[cIdx];
        ctx.shadowColor = COLORS[(cIdx+2)%COLORS.length];
        ctx.shadowBlur = 10 + 18*Math.abs(Math.sin(t+ring));
        ctx.globalAlpha = alpha;

        ctx.lineWidth = 2.5 + 1.5*Math.sin(tRing*1.3 + ring*0.77);

        ctx.stroke();
    }
    ctx.globalAlpha = 1;
    ctx.restore();

    // Subtle central circle pulse, with radiating bands
    for(let i=0;i<3;++i) {
        let r = 24 + 10*i + 6*Math.sin(time*2.3+i);
        ctx.beginPath();
        ctx.arc(cx,cy,r,0,2*Math.PI);
        ctx.strokeStyle = COLORS[(i*4+Math.floor(time*2.5))%COLORS.length];
        ctx.globalAlpha = 0.17 + 0.13*i;
        ctx.lineWidth = 2.1 - 0.3*i;
        ctx.shadowBlur = 12-4*i;
        ctx.stroke();
    }
    ctx.globalAlpha = 1;
    ctx.shadowBlur = 0;
}

function animate() {
    time += 0.02*speed;
    drawSpiralingWavefield(ctx, time, {waveCount});
    requestAnimationFrame(animate);
}

// Responsive
window.addEventListener('resize', ()=>{
    let dpr = window.devicePixelRatio || 1;
    let side = Math.min(window.innerWidth, window.innerHeight) - 60;
    side = Math.max(350, Math.min(800, side));
    canvas.width = side*dpr;
    canvas.height = side*dpr;
    canvas.style.width = side+"px";
    canvas.style.height = side+"px";
    ctx.setTransform(1,0,0,1,0,0);
    ctx.scale(dpr, dpr);
});
window.dispatchEvent(new Event('resize'));

animate();
</script>
