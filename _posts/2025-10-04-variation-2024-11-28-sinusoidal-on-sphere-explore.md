---
layout: fullscreen
title: "Psychedelic Lissajous Dust"
tags:
  - graphics
---

<style>
    canvas {
        background: radial-gradient(ellipse at center, #000 0%, #222 100%);
        display: block;
        margin: 0 auto 20px auto;
        border: 2px solid #444;
    }
    .controls {
        display: grid;
        grid-template-columns: auto 1fr auto;
        gap: 10px;
        align-items: center;
        width: 90%;
        margin: 0 auto 15px auto;
        color: #eee;
    }
    .footer-link {
        position: absolute;
        bottom: 20px;
        text-align: center;
        width: 100%;
    }
    .footer-link a {
        font-size: 16px;
        color: #05f2db;
        text-decoration: none;
    }
    .footer-link a:hover {
        text-decoration: underline;
    }
</style>
<canvas id="dustCanvas" width="900" height="900"></canvas>

<div class="controls">
    <label for="cx">X Freq:</label>
    <input type="range" id="cx" min="1" max="12" value="5" step="1">
    <span id="cxValue">5</span>

    <label for="cy">Y Freq:</label>
    <input type="range" id="cy" min="1" max="12" value="8" step="1">
    <span id="cyValue">8</span>

    <label for="pmod">Phase:</label>
    <input type="range" id="pmod" min="0" max="628" value="0" step="1">
    <span id="pmodValue">0</span>

    <label for="spread">Spread:</label>
    <input type="range" id="spread" min="80" max="440" value="260" step="1">
    <span id="spreadValue">260</span>

    <label for="nbands">Bands:</label>
    <input type="range" id="nbands" min="1" max="16" value="8" step="1">
    <span id="nbandsValue">8</span>

    <label for="dust">Dust:</label>
    <input type="range" id="dust" min="1" max="60" value="20" step="1">
    <span id="dustValue">20</span>
</div>

<script>
const canvas = document.getElementById('dustCanvas');
const ctx = canvas.getContext('2d');
const width = canvas.width;
const height = canvas.height;
const centerX = width / 2;
const centerY = height / 2;

// Controls
let cx = 5, cy = 8, pmod = 0, spread = 260, nbands = 8, dust = 20;

function updateValuesFromControls() {
    cx = parseInt(document.getElementById('cx').value);
    cy = parseInt(document.getElementById('cy').value);
    pmod = parseInt(document.getElementById('pmod').value) / 100;
    spread = parseInt(document.getElementById('spread').value);
    nbands = parseInt(document.getElementById('nbands').value);
    dust = parseInt(document.getElementById('dust').value);

    document.getElementById('cxValue').textContent = cx;
    document.getElementById('cyValue').textContent = cy;
    document.getElementById('pmodValue').textContent = Math.round(pmod*100);
    document.getElementById('spreadValue').textContent = spread;
    document.getElementById('nbandsValue').textContent = nbands;
    document.getElementById('dustValue').textContent = dust;
}
updateValuesFromControls();

for (let control of ["cx","cy","pmod","spread","nbands","dust"]) {
    document.getElementById(control).addEventListener('input', () => {
        updateValuesFromControls();
        render = true;
    });
}

// Lissajous Dust Animation Variables
const totalParticles = 550;
let dustStates = [];
let t = 0;
let render = true;

function resetDustStates() {
    dustStates = [];
    for (let i = 0; i < totalParticles; i++) {
        dustStates.push({
            px: Math.random()*width,
            py: Math.random()*height,
            phase: Math.random()*Math.PI*2,
            offset: (i/totalParticles) * Math.PI*2,
        });
    }
}
resetDustStates();

function hsv2rgb(h, s, v) {
    let f = (n, k = (h/60 + n) % 6) => v - v*s*Math.max(Math.min(k,4-k,1),0);
    return [f(5)*255, f(3)*255, f(1)*255];
}

function draw() {
    // Trailing effect
    ctx.globalAlpha = 0.24;
    ctx.fillStyle = "#000";
    ctx.fillRect(0, 0, width, height);
    ctx.globalAlpha = 1.0;

    let freq_x = cx, freq_y = cy;
    let phase = t*0.013 + pmod;
    let modbands = nbands * 1.2 + 1; // add some aliasing at high bands

    for (let i = 0; i < totalParticles; i++) {
        let band = i % nbands;
        let dustfreq = 1 + band/modbands;
        let theta = (t*0.005 + dustStates[i].phase + dustStates[i].offset * dustfreq);

        // Lissajous XY
        let lx = Math.sin(freq_x * theta + Math.sin(theta+phase));
        let ly = Math.sin(freq_y * theta + Math.cos(theta-phase)*1.12);

        // Spiral spread modulated like a flower
        let rad = spread * (0.83 + 0.22*Math.sin(band/nbands*Math.PI*8 + phase*0.7));
        let spiral = (theta + Math.sin(theta*0.55+band))*0.33;

        let px = centerX + lx * rad * Math.cos(spiral) - ly * rad * Math.sin(spiral);
        let py = centerY + lx * rad * Math.sin(spiral) + ly * rad * Math.cos(spiral);

        // "dust" swarming, small random walk
        dustStates[i].px += (px - dustStates[i].px) * 0.12 + (Math.random()-0.5)*dust*0.04;
        dustStates[i].py += (py - dustStates[i].py) * 0.12 + (Math.random()-0.5)*dust*0.04;

        // Psychedelic color sweep
        let h = ((theta*30 + t*0.7 + band*42) % 360 + 360) % 360;
        let s = 0.93 - 0.4*Math.sin(theta+band);
        let v = 0.92 - 0.15*Math.cos(theta*0.8+band+phase);
        let [r,g,b] = hsv2rgb(h, s, v);

        // Main "dust" particle
        ctx.beginPath();
        ctx.arc(dustStates[i].px, dustStates[i].py, 2.8 + 1.5*Math.sin(theta+phase+band), 0, Math.PI*2);
        ctx.fillStyle = `rgba(${Math.floor(r)},${Math.floor(g)},${Math.floor(b)},0.57)`;
        ctx.shadowColor = `rgba(${Math.floor(r)},${Math.floor(g)},${Math.floor(b)},0.43)`;
        ctx.shadowBlur = 14 + 17*Math.abs(Math.sin(phase+band));
        ctx.fill();
    }

    ctx.shadowBlur = 0;
}

function animate() {
    if (render) {
        draw();
        render = false;
    }
    t += 1;
    requestAnimationFrame(animate);
}
animate();

// Redraw on parameter changes
setInterval(() => { render = true; }, 100);

// Re-randomize on double-click
canvas.addEventListener("dblclick", () => {
    resetDustStates();
    render = true;
});
</script>

<div class="footer-link">
  Double-click the canvas to morph the swarm.<br>
  <a href="https://en.wikipedia.org/wiki/Lissajous_curve" target="_blank">About Lissajous curves</a>
</div>
