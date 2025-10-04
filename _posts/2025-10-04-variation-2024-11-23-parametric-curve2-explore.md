```markdown
---
layout: fullscreen
title: "Psychedelic Lissajous Particle Swarm - Trippy Trails"
tags:
  - graphics
---

<a href="https://tthtlc.wordpress.com/2024/10/18/psychedelic-lissajous-particle-swarm/">Psychedelic Lissajous Particle Swarm</a> <br>

<style>
    canvas {
        background: radial-gradient(ellipse at center, #110019 0%, #281a4c 100%);
        border: 1px solid #444;
        display: block;
        margin: 0 auto;
        box-shadow: 0 0 32px #0008;
    }
    .controls {
        position: absolute;
        bottom: 32px;
        width: 800px;
        left: 50%;
        transform: translateX(-50%);
        display: flex;
        justify-content: center;
        gap: 24px;
        z-index: 2;
    }
    .control-group {
        display: flex;
        flex-direction: column;
        align-items: center;
        color: #ddd;
        font-size: 14px;
    }
</style>

<canvas id="canvas" width="800" height="800"></canvas>
<br><br>
<div class="controls">
    <div class="control-group">
        <label for="freqX">Lissajous freq X:</label>
        <input type="range" id="freqX" min="1" max="12" step="0.1" value="7.4">
        <span id="value-freqX">7.4</span>
    </div>
    <div class="control-group">
        <label for="freqY">Lissajous freq Y:</label>
        <input type="range" id="freqY" min="1" max="12" step="0.1" value="9.2">
        <span id="value-freqY">9.2</span>
    </div>
    <div class="control-group">
        <label for="swarm">Swarm size:</label>
        <input type="range" id="swarm" min="10" max="120" step="1" value="55">
        <span id="value-swarm">55</span>
    </div>
</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const center = {x: W/2, y: H/2};

let freqX = parseFloat(document.getElementById('freqX').value);
let freqY = parseFloat(document.getElementById('freqY').value);
let swarm = parseInt(document.getElementById('swarm').value);

const valueFreqX = document.getElementById('value-freqX');
const valueFreqY = document.getElementById('value-freqY');
const valueSwarm = document.getElementById('value-swarm');

// Controls update
function updateValueDisplay(id, value) {
    document.getElementById(`value-${id}`).textContent = value;
}
document.getElementById('freqX').addEventListener('input', e => {
    freqX = parseFloat(e.target.value);
    updateValueDisplay('freqX', freqX);
});
document.getElementById('freqY').addEventListener('input', e => {
    freqY = parseFloat(e.target.value);
    updateValueDisplay('freqY', freqY);
});
document.getElementById('swarm').addEventListener('input', e => {
    swarm = parseInt(e.target.value);
    updateValueDisplay('swarm', swarm);
    resetSwarm();
});

// Lissajous-perturbed particles with trippy trails
let particles = [];
const TAU = Math.PI * 2;

// Colors: cycle through psychedelic palette
function hsl(h, s, l, a=1) {
    return `hsla(${h},${s}%,${l}%,${a})`;
}

function Particle(index) {
    this.i = index;
    this.θOff = Math.random() * TAU;
    this.ϕOff = Math.random() * TAU;
    this.baseR = 170 + 80 * Math.sin(index * 1.2);
    this.dR = 43 + 28 * Math.cos(index*0.8);
    this.colorOffset = Math.random() * 360;
    this.trail = [];
    this.maxTrail = 26 + Math.floor(Math.random()*8);
}
Particle.prototype.pos = function(t) {
    // Evolving lissajous path, with "breathing" radial perturbation
    const θ = t * 0.38 + this.θOff;
    const ϕ = t * 0.33 + this.ϕOff;

    const x = Math.sin(freqX * θ + this.θOff + 0.9 * Math.cos(t*0.18 + this.i));
    const y = Math.sin(freqY * ϕ + this.ϕOff - 0.8 * Math.cos(t*0.23 - this.i));

    // Time-evolving "radial tremor" for inner/middle ring structure
    const r = this.baseR + this.dR * Math.sin(t*0.11 + this.i) + 12*Math.sin(t*0.9 + this.i);

    // Add polar morphic 'pulse'
    const q = 1 + 0.23 * Math.sin(t*0.41 + this.i);

    return {
        x: center.x + r * x * q,
        y: center.y + r * y * q
    };
}
Particle.prototype.clr = function(t) {
    // Hues sweep with time and ring
    return hsl(
        (t*20 + this.colorOffset + 57*this.i)%360,
        82,
        54 + 11*Math.sin(t*0.08 + this.i),
        0.95
    );
}
function resetSwarm() {
    particles = [];
    for (let i=0;i<swarm;i++) particles.push(new Particle(i));
}
resetSwarm();

// Animation loop
let t = 0, last = performance.now();

function draw() {
    // Faint trailing, not full clear (feedback)
    ctx.save();
    ctx.globalAlpha = 0.13;
    ctx.fillStyle = "#14001b";
    ctx.fillRect(0,0,W,H);
    ctx.restore();

    // Optionally: overlay a radial glow (subtle, for halo vibes)
    for (let i=0;i<3;i++) {
        ctx.save();
        ctx.globalAlpha = 0.08 - 0.015*i;
        const g = ctx.createRadialGradient(
            center.x, center.y, 50+20*i,
            center.x, center.y, W/2.1
        );
        g.addColorStop(0, "#5ef1a0");
        g.addColorStop(0.3 + i*0.14, "#55e1d7");
        g.addColorStop(1, "rgba(0,0,0,0)");
        ctx.fillStyle = g;
        ctx.beginPath();
        ctx.arc(center.x, center.y, W/2.05, 0, TAU);
        ctx.fill();
        ctx.restore();
    }

    // Animate swarm particles
    for (let p of particles) {
        // Current and previous positions for trail
        const pos = p.pos(t);
        p.trail.unshift(pos);
        if (p.trail.length > p.maxTrail) p.trail.pop();

        // Trail (fades to back, psychedelic color drift)
        for (let j=p.trail.length-1; j>0; j--) {
            const alpha = (1 - j/p.trail.length) * 0.77;
            ctx.strokeStyle = hsl(
                (t*20 + p.colorOffset + 57*p.i + j*4)%360,
                88,
                57 + 11*Math.sin(t*0.08 + p.i + j*0.04),
                alpha
            );
            ctx.lineWidth = 2.6 - 2*j/p.trail.length;
            ctx.beginPath();
            ctx.moveTo(p.trail[j].x, p.trail[j].y);
            ctx.lineTo(p.trail[j-1].x, p.trail[j-1].y);
            ctx.stroke();
        }

        // Head (dot)
        ctx.save();
        ctx.globalAlpha = 0.95;
        ctx.beginPath();
        ctx.arc(pos.x, pos.y, 3.4 + 2*Math.sin(t*0.4 + p.i), 0, TAU);
        ctx.fillStyle = p.clr(t);
        ctx.shadowColor = p.clr(t);
        ctx.shadowBlur = 9 + 3*Math.sin(t*0.91 + p.i);
        ctx.fill();
        ctx.restore();
    }
}

function animate(now) {
    let dt = Math.min((now-last)/1000, 0.08);
    t += dt * 1.11; // time step
    last = now;
    draw();
    requestAnimationFrame(animate);
}
animate(performance.now());

</script>
```