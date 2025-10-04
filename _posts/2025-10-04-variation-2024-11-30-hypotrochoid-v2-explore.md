---
layout: fullscreen
title: "Psychedelic Auroras: Animated Lissajous Particle Waves"
tags:
  - graphics
---

<style>
    canvas {
        display: block;
        margin: 0 auto;
        border: 1px solid #111;
        background: radial-gradient(ellipse at center, #181e30 0%, #24243e 100%);
    }

    /* Desktop styles */
    @media screen and (min-width: 768px) {
        canvas {
            width: 100vw;
            height: 100vh;
            max-width: 1200px;
            max-height: 1200px;
        }
    }

    /* Mobile styles */
    @media screen and (max-width: 767px) {
        canvas {
            width: 98vw;
            height: 98vw;
            max-width: 540px;
            max-height: 540px;
        }
    }

    .controls {
        margin-top: 20px;
        display: flex;
        flex-direction: column;
        align-items: center;
        position: absolute;
        left: 0;
        right: 0;
        top: 15px;
        z-index: 20;
        pointer-events: none;
    }
    .control-group {
        margin: 10px 0 0 0;
        display: flex;
        align-items: center;
        font-size: 1.08em;
        background: rgba(30,30,40,0.8);
        border-radius: 5px;
        padding: 5px 18px;
        pointer-events: auto;
    }
    .control-group label {
        margin-right: 10px;
        color: #d0def7;
        font-family: monospace;
    }
    input[type="range"] {
        width: 140px;
        accent-color: #a6ffcb;
    }
    .value-label {
        margin-left: 14px;
        font-weight: bold;
        color: #76e1ff;
        font-family: monospace;
    }
    @media (max-width: 700px) {
        .control-group label {
            font-size: 15px;
        }
        .control-group {
            font-size: 13px;
            padding: 3px 6px;
        }
    }
</style>

<canvas id="canvas"></canvas>

<div class="controls">
    <div class="control-group">
        <label for="particles">Particles:</label>
        <input type="range" id="particles" min="80" max="420" value="180">
        <span id="particles-value" class="value-label">180</span>
    </div>
    <div class="control-group">
        <label for="frequencyX">Freq X:</label>
        <input type="range" id="frequencyX" min="1" max="8" value="4">
        <span id="frequencyX-value" class="value-label">4</span>
    </div>
    <div class="control-group">
        <label for="frequencyY">Freq Y:</label>
        <input type="range" id="frequencyY" min="1" max="8" value="5">
        <span id="frequencyY-value" class="value-label">5</span>
    </div>
    <div class="control-group">
        <label for="colorSpread">Color Spread:</label>
        <input type="range" id="colorSpread" min="40" max="360" value="180">
        <span id="colorSpread-value" class="value-label">180</span>
    </div>
    <div class="control-group">
        <label for="trail">Trails:</label>
        <input type="range" id="trail" min="5" max="90" value="30">
        <span id="trail-value" class="value-label">30</span>
    </div>
</div>

<script>
/**
 * Psychedelic Auroras: Animated Lissajous Particle Waves
 * ~ By Generative Artist AI
 * self-contained, interactive canvas art
 */

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let width, height, cx, cy;

// PARAMETERS (default)
let particleCount = 180;
let fx = 4;
let fy = 5;
let colorSpread = 180; // in degrees
let trailStrength = 30; // alpha for motion trails

// Responsive canvas setup
function resizeCanvas() {
    // Use device pixel ratio for crisp rendering
    const dpr = window.devicePixelRatio || 1;
    const rect = canvas.getBoundingClientRect();
    width = rect.width;
    height = rect.height;
    canvas.width = width * dpr;
    canvas.height = height * dpr;
    ctx.setTransform(1,0,0,1,0,0); // reset
    ctx.scale(dpr, dpr);

    cx = width / 2;
    cy = height / 2;
}
resizeCanvas();
window.addEventListener('resize', () => {
    resizeCanvas();
});

// Interactive controls logic
function updateControl(id, v) {
    document.getElementById(id + '-value').innerText = v;
}

document.getElementById('particles').addEventListener('input', function() {
    particleCount = parseInt(this.value);
    updateControl('particles', this.value);
});
document.getElementById('frequencyX').addEventListener('input', function() {
    fx = parseInt(this.value);
    updateControl('frequencyX', this.value);
});
document.getElementById('frequencyY').addEventListener('input', function() {
    fy = parseInt(this.value);
    updateControl('frequencyY', this.value);
});
document.getElementById('colorSpread').addEventListener('input', function() {
    colorSpread = parseInt(this.value);
    updateControl('colorSpread', this.value);
});
document.getElementById('trail').addEventListener('input', function() {
    trailStrength = parseInt(this.value);
    updateControl('trail', this.value);
});

// Particle aurora engine
function lerp(a, b, t) { return a + (b-a)*t; }

function drawAuroraParticles(time) {
    // Fade out old frame for motion trails
    ctx.globalCompositeOperation = 'destination-out';
    ctx.fillStyle = `rgba(24,30,48,${trailStrength/100})`;
    ctx.fillRect(0, 0, width, height);

    ctx.globalCompositeOperation = 'lighter';

    // Dynamic radius based on canvas size
    const R = Math.min(width, height) * 0.42;
    // Slowly morphing orbit amplitudes for psychedelic effect
    const ampX = lerp(R * 0.75, R * 1.08, 0.4 + 0.36*Math.sin(time * 0.12));
    const ampY = lerp(R * 0.65, R * 1.03, 0.6 + 0.30*Math.cos(time * 0.10));
    const wavePhase = Math.sin(time * 0.06) * Math.PI;

    // Loop through all particles
    for(let i=0; i<particleCount; i++) {
        // Particle-specific phase offset for swirling effect
        const p = i/particleCount;
        // Animate phase with slow offset
        const t = time/700 + p*4 + Math.cos(time/1200 + i*0.069)*0.34;

        // Lissajous paths
        const angleX = fx * t + wavePhase;
        const angleY = fy * t;

        const x = cx + ampX * Math.sin(angleX + Math.sin(i*0.07 + time*0.0004)*0.4);
        const y = cy + ampY * Math.cos(angleY + Math.cos(i*0.031 - time*0.0008)*0.5);

        // Particle color: rainbow + aurora gradient
        // Hue rotates along the wave, animated
        let hue = ((p*colorSpread) + time*0.07 + Math.sin(i*0.13 + time*0.0006)*66) % 360;
        // Map a greenish-pink bias, aurora-style
        const auroraRatio = Math.abs(Math.sin(angleX*0.31 + angleY*0.29));
        let r = lerp(60, 110, auroraRatio);
        let g = lerp(220, 70, auroraRatio);
        let b = lerp(180, 250, 1-auroraRatio);

        // Use HSL for base, overlay aurora color blend
        ctx.save();
        ctx.beginPath();
        ctx.arc(x, y, lerp(2.3, 4.2, Math.cos(i + time*0.003)*0.5+0.5), 0, Math.PI*2);

        // Vibrant neon glow
        const grad = ctx.createRadialGradient(x, y, 1, x, y, 13);
        grad.addColorStop(0, `hsla(${hue}, 95%, 65%, 0.19)`);
        grad.addColorStop(0.5, `rgba(${Math.round(r)},${Math.round(g)},${Math.round(b)},0.30)`);
        grad.addColorStop(1, 'rgba(10,18,33,0)');
        ctx.fillStyle = grad;
        ctx.shadowColor = `hsl(${Math.round(hue)},95%, 75%)`;
        ctx.shadowBlur = 13;

        ctx.strokeStyle = `hsl(${Math.round(hue)},92%,92%)`;
        ctx.lineWidth = 0.54 + 0.4*Math.abs(Math.sin(i*1.31 + time*0.002));

        ctx.fill();
        ctx.stroke();
        ctx.restore();
    }
}

// Animation loop
let lastWidth = 0, lastHeight = 0;
function animate(ts) {
    // If size changed, update
    const rect = canvas.getBoundingClientRect();
    if(rect.width !== lastWidth || rect.height !== lastHeight) {
        resizeCanvas();
        lastWidth = rect.width;
        lastHeight = rect.height;
    }

    drawAuroraParticles(performance.now());

    requestAnimationFrame(animate);
}
animate();

</script>
