---
layout: fullscreen
title: Psychedelic Pulsing Wave Portal
tags:
  - graphics
---

<canvas id="animationCanvas" width="600" height="600"></canvas>
<div class="controls">
    <label for="waveCount">Wave count:</label>
    <input type="range" id="waveCount" name="waveCount" min="2" max="12" step="1" value="6">
    <span id="waveCountValue">6</span>
    <button id="startStopButton">Start</button>
</div>
<script>
const canvas = document.getElementById('animationCanvas');
const ctx = canvas.getContext('2d');
const centerX = canvas.width / 2;
const centerY = canvas.height / 2;
const baseRadius = 80;
const waveRadius = 200;
let waveCount = 6;
let isAnimating = false;
let animationFrameId;
let t = 0;

const waveCountSlider = document.getElementById('waveCount');
const waveCountValueDisplay = document.getElementById('waveCountValue');
const startStopButton = document.getElementById('startStopButton');

waveCountSlider.addEventListener('input', (event) => {
    waveCount = parseInt(event.target.value);
    waveCountValueDisplay.textContent = waveCount;
});

function lerp(a, b, f) {
    return a + (b - a) * f;
}

// Generate a trippy, seamless color palette
function getPsyColor(theta, offset=0) {
    let freq = 0.3;
    let r = Math.sin(freq * theta + 0 + offset) * 127 + 128;
    let g = Math.sin(freq * theta + 2 + offset) * 127 + 128;
    let b = Math.sin(freq * theta + 4 + offset) * 127 + 128;
    return `rgb(${r|0},${g|0},${b|0})`;
}

// Draw one full warped circle using multiple superimposed waveforms
function drawWavePortal(waveCount, t) {
    for (let ring = 6; ring >= 0; ring--) {
        ctx.save();
        // Shrink/quiver every ring for depth
        let phaseOffset = t * 0.8 + ring * Math.PI/8;
        let radiusRing = lerp(waveRadius, baseRadius, ring / 7.0) * (1 + Math.sin(t*1.7 - ring*0.07 + Math.sin(t*1.1+ring)) * 0.065);
        ctx.beginPath();
        for(let i = 0; i <= 360; i += 1) {
            const theta = i * Math.PI / 180;
            // Number of "petals"/waves increases for inner rings
            let numWaves = waveCount + ring;
            // Phase and strength vary per ring
            let amp = 18 + 8*Math.sin(t*0.45+ring);
            let wavy = Math.sin(numWaves * theta + phaseOffset) * amp;
            let r = radiusRing + wavy;
            let x = centerX + Math.cos(theta) * r;
            let y = centerY + Math.sin(theta) * r;
            if(i === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        // Psychedelic fill
        ctx.closePath();
        ctx.globalAlpha = lerp(0.33, 0.92, ring / 7.0); // Outer rings are more translucent
        ctx.fillStyle = getPsyColor(t*2 + ring*1.4, ring*0.5);
        ctx.shadowColor = getPsyColor(t*1.6 + ring*3, ring);
        ctx.shadowBlur = 26-(ring*2.2);
        ctx.fill();
        ctx.restore();
    }
}

// Sine-circle particles orbiting and leaving pulse trails
const PARTICLE_COUNT = 80;
let particles = [];
function initParticles() {
    particles = [];
    for(let i=0; i<PARTICLE_COUNT; i++) {
        let angle = Math.random()*Math.PI*2;
        let speed = lerp(0.012, 0.033, Math.random());
        let orbit = lerp(baseRadius+28, waveRadius-12, Math.random());
        let phase = Math.random()*Math.PI*2;
        particles.push({angle, speed, orbit, phase});
    }
}
initParticles();

function drawParticles(t) {
    for(let i=0; i<particles.length; i++) {
        let p = particles[i];
        let theta = p.angle + t*p.speed + Math.sin(t*0.7+p.phase)*0.4;
        // Orbit pulse
        let orbitPulse = p.orbit + Math.sin(t*1.4+p.phase*3)*11;
        // Slight "wiggle"
        let x = centerX + Math.cos(theta)*orbitPulse + Math.cos(t+p.phase*6)*3;
        let y = centerY + Math.sin(theta)*orbitPulse + Math.sin(t+p.phase*4)*3;
        // Particle color: animated rainbow 
        ctx.save();
        ctx.globalAlpha = 0.67 + 0.33 * Math.sin(t*2+p.phase);
        ctx.beginPath();
        ctx.arc(x, y, 5 + 1.5*Math.sin(t*3 + i), 0, Math.PI*2);
        ctx.fillStyle = getPsyColor(theta*5 + t*1.1 + p.phase*3, t*.3 - i);
        ctx.shadowColor = getPsyColor(theta*4 + t*2, t*.8 - i*1.2);
        ctx.shadowBlur = 16;
        ctx.fill();
        ctx.restore();
        // Trailing
        if (Math.random() < 0.17) {
            ctx.save();
            ctx.globalAlpha = 0.09 + 0.11 * Math.sin(t*1.2+i);
            ctx.beginPath();
            ctx.arc(x, y, 7 + 1.4*Math.sin(p.phase+t), 0, Math.PI*2);
            ctx.fillStyle = getPsyColor(theta*3 - t*2.6, p.phase*2);
            ctx.shadowColor = getPsyColor(theta*2 - t*2.2, t*.7);
            ctx.shadowBlur = 12;
            ctx.fill();
            ctx.restore();
        }
    }
}

// Trippy background fade
function fadeBackground() {
    ctx.save();
    ctx.globalAlpha = 0.105;
    ctx.fillStyle = "#120e24";
    ctx.fillRect(0,0,canvas.width,canvas.height);
    ctx.restore();
}

function animate() {
    if(!isAnimating) return;
    t += 0.022;
    fadeBackground();
    drawWavePortal(waveCount, t);
    drawParticles(t);
    animationFrameId = requestAnimationFrame(animate);
}

startStopButton.addEventListener('click', () => {
    isAnimating = !isAnimating;
    if(isAnimating) {
        startStopButton.textContent = "Stop";
        // On Start, refresh the visuals
        fadeBackground();
        initParticles();
        animate();
    } else {
        startStopButton.textContent = "Start";
        cancelAnimationFrame(animationFrameId);
    }
});

// Draw initial frame
fadeBackground();
drawWavePortal(waveCount, t);
drawParticles(t);
</script>
