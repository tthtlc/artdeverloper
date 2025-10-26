---
layout: fullscreen
title: "Psychedelic Sinusoidal Orbital Web"
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
        margin: 12px 0;
        font-family: Arial, sans-serif;
    }
    .controls label {
        margin-right: 12px;
    }
</style>
<div class="controls">
    <label for="waveCount">Waves: <span id="waveCountValue">5</span></label>
    <input type="range" id="waveCount" min="2" max="16" step="1" value="5">

    <label for="particleCount">Particles: <span id="particleCountValue">180</span></label>
    <input type="range" id="particleCount" min="60" max="360" step="10" value="180">

    <label for="pulseSpeed">Pulse: <span id="pulseSpeedValue">0.012</span></label>
    <input type="range" id="pulseSpeed" min="1" max="50" step="1" value="12">

    <label for="webDensity">Web: <span id="webDensityValue">3</span></label>
    <input type="range" id="webDensity" min="1" max="10" step="1" value="3">
</div>
<canvas id="animationCanvas" width="800" height="800"></canvas>
<script>
    // Setup and controls
    const canvas = document.getElementById('animationCanvas');
    const ctx = canvas.getContext('2d');

    const waveCount = document.getElementById('waveCount');
    const waveCountValue = document.getElementById('waveCountValue');
    let nWaves = parseInt(waveCount.value);

    const particleCount = document.getElementById('particleCount');
    const particleCountValue = document.getElementById('particleCountValue');
    let nParticles = parseInt(particleCount.value);

    const pulseSpeed = document.getElementById('pulseSpeed');
    const pulseSpeedValue = document.getElementById('pulseSpeedValue');
    let pSpeed = parseInt(pulseSpeed.value) * 0.001;

    const webDensity = document.getElementById('webDensity');
    const webDensityValue = document.getElementById('webDensityValue');
    let wDensity = parseInt(webDensity.value);

    waveCount.addEventListener('input', () => {
        nWaves = parseInt(waveCount.value);
        waveCountValue.textContent = nWaves;
    });
    particleCount.addEventListener('input', () => {
        nParticles = parseInt(particleCount.value);
        particleCountValue.textContent = nParticles;
    });
    pulseSpeed.addEventListener('input', () => {
        pSpeed = parseInt(pulseSpeed.value) * 0.001;
        pulseSpeedValue.textContent = (pSpeed).toFixed(3);
    });
    webDensity.addEventListener('input', () => {
        wDensity = parseInt(webDensity.value);
        webDensityValue.textContent = wDensity;
    });

    // Utility HSV to RGB for psychedelic coloring
    function hsvToRgb(h, s, v) {
        let f = (n, k = (h / 60 + n) % 6) =>
            v - v * s * Math.max(Math.min(k, 4 - k, 1), 0);
        return [f(5), f(3), f(1)];
    }

    // Generates points orbiting the center with superimposed sinusoidal modulations
    function getOrbitalPoints(time) {
        const cx = canvas.width / 2;
        const cy = canvas.height / 2;
        const R0 = Math.min(cx, cy) * 0.46;
        let points = [];
        for (let i = 0; i < nParticles; i++) {
            let theta = (2 * Math.PI * i) / nParticles;

            // Compose multiple "wave" ripples for the orbit
            let r = R0;
            for (let k = 1; k <= nWaves; k++) {
                let freq = k;
                let mag = R0 * (0.11 / k);
                r += Math.sin(
                    freq * theta
                    + 1.1 * Math.sin(time / (1.7 + k * 0.13) + i * 0.19)
                ) * mag * Math.sin(time * (1 + k * 0.09));
            }

            // Add pulsing
            let pulse = (0.94 + 0.18 * Math.sin(time * 0.8 + i * 0.11));
            r *= pulse;

            let x = cx + r * Math.cos(theta);
            let y = cy + r * Math.sin(theta);

            points.push({x, y, theta, idx: i});
        }
        return points;
    }

    // Draw background with fading trails (additive)
    function fadeBackground() {
        ctx.globalCompositeOperation = "lighter";
        ctx.fillStyle = "rgba(0,0,0,0.14)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.globalCompositeOperation = "source-over";
    }

    // Draw colored connected web
    function drawWeb(points, time) {
        ctx.save();
        ctx.lineWidth = 1.32 + 0.38*Math.sin(time/11);
        for (let i = 0; i < points.length; i++) {
            // Hues based on position and time for rainbow
            let hueA = (points[i].theta / (2 * Math.PI)) * 360 + 85 * Math.sin(time/8 + i*0.07);
            let sat = 0.7 + 0.25 * Math.sin(time * 0.4 + i * 0.10);
            let val = 0.93;
            let [rA, gA, bA] = hsvToRgb(hueA, sat, val);

            for (let k = 1; k <= wDensity; k++) {
                let j = (i + Math.floor(nParticles / wDensity) * k) % points.length;
                // Hue between points for color blending
                let hueB = (points[j].theta / (2 * Math.PI)) * 360 + 85 * Math.cos(time/7 + j*0.055);
                let [rB, gB, bB] = hsvToRgb(hueB, sat, val);
                ctx.strokeStyle = `rgba(
                    ${Math.floor(0.6*rA + 0.4*rB)},
                    ${Math.floor(0.6*gA + 0.4*gB)},
                    ${Math.floor(0.6*bA + 0.4*bB)},
                    0.20
                )`;
                ctx.beginPath();
                ctx.moveTo(points[i].x, points[i].y);
                ctx.lineTo(points[j].x, points[j].y);
                ctx.stroke();
            }
        }
        ctx.restore();
    }

    // "Comet" particle rendering at each point (psychedelic auras)
    function drawParticles(points, time) {
        for (const pt of points) {
            ctx.save();
            let aura = 1 + 0.46 * Math.sin(time*1.4 + pt.idx * 0.161);
            let h = (pt.theta/(2*Math.PI))*360 + 170*Math.sin(pt.idx*0.05 + time/3);
            let [r, g, b] = hsvToRgb(h, 0.85, 1.0);
            let alpha = 0.47 + 0.34 * Math.sin(time*0.7 + pt.idx*0.09);

            let rad = 7.5 + 5 * aura;
            let grad = ctx.createRadialGradient(pt.x, pt.y, 1, pt.x, pt.y, rad);
            grad.addColorStop(0, `rgba(${r},${g},${b},${alpha})`);
            grad.addColorStop(0.65, `rgba(${r},${g},${b},${alpha*0.23})`);
            grad.addColorStop(1, 'rgba(0,0,0,0)');
            ctx.globalAlpha = 0.91;
            ctx.beginPath();
            ctx.arc(pt.x, pt.y, rad, 0, 2*Math.PI);
            ctx.fillStyle = grad;
            ctx.fill();
            ctx.restore();
        }
    }

    // Main animation loop
    let t = 0;
    function loop() {
        fadeBackground();
        let points = getOrbitalPoints(t);

        drawWeb(points, t);
        drawParticles(points, t);

        t += pSpeed;
        requestAnimationFrame(loop);
    }

    // Resize canvas to fit window (fullscreen)
    function resizeCanvas() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();

    // Start!
    ctx.globalAlpha = 1.0;
    ctx.fillStyle = "black";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    loop();
</script>
