---
layout: fullscreen
title: "Sinusoidal Pulsar Mandala"
tags:
  - graphics
---

<canvas id="mandalaCanvas" width="600" height="600"></canvas>
<script>
// Sinusoidal Pulsar Mandala - Psychedelic Generative Animation

const canvas = document.getElementById('mandalaCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width;
const H = canvas.height;

const CENTER = { x: W / 2, y: H / 2 };
const NUM_ARMS = 12;
const NUM_WAVES = 8;
const BASE_RADIUS = 80;
const MOD_RADIUS = 110;
const PULSE_SPEED = 0.021;
const COLOR_CYCLE_SPEED = 0.025;
const PARTICLE_TRAIL = 0.13;

// Generates a smoothly cycling rainbow palette
function getRainbowColor(t, alpha=1) {
    const r = Math.round(128 + 128 * Math.sin(2 * Math.PI * (t + 0.0)));
    const g = Math.round(128 + 128 * Math.sin(2 * Math.PI * (t + 0.333)));
    const b = Math.round(128 + 128 * Math.sin(2 * Math.PI * (t + 0.666)));
    return `rgba(${r},${g},${b},${alpha})`;
}

let time = 0;

// For a trippy trail effect
function fadeCanvas() {
    ctx.globalAlpha = PARTICLE_TRAIL;
    ctx.fillStyle = "#0a0120";
    ctx.fillRect(0, 0, W, H);
    ctx.globalAlpha = 1.0;
}

function drawSinusoidalMandala(t) {
    for (let arm = 0; arm < NUM_ARMS; arm++) {
        const baseAngle = (2 * Math.PI / NUM_ARMS) * arm;
        const colorT = (t * COLOR_CYCLE_SPEED + arm / NUM_ARMS) % 1;
        const strokeColor = getRainbowColor(colorT, 0.75);

        ctx.beginPath();
        for (let wave = 0; wave <= NUM_WAVES; wave++) {
            // Angle along this arm with minor rotation
            let angle =
                baseAngle
                + wave * Math.PI / NUM_WAVES
                + 0.35 * Math.sin(t * 0.4 + arm + wave);

            // Radial displacement with several modulations
            let r =
                BASE_RADIUS
                + MOD_RADIUS * Math.abs(Math.sin(t * PULSE_SPEED + arm * 0.9 + wave))
                + 22 * Math.sin(wave + t + arm * 1.3)
                + 17 * Math.cos(wave * 2.1 - t * 0.7 + arm);

            // Central pulsating flower
            if (wave === 0) {
                r *= 0.94 + 0.12 * Math.cos(arm + t * 1.1 + wave * 2);
            }

            const x = CENTER.x + r * Math.cos(angle);
            const y = CENTER.y + r * Math.sin(angle);

            if (wave === 0)
                ctx.moveTo(x, y);
            else
                ctx.lineTo(x, y);

            // Draw flickering nodes at peaks
            if (wave > 0 && Math.abs(Math.sin(t + wave + arm)) > 0.96) {
                ctx.save();
                ctx.strokeStyle = "#fff";
                ctx.globalAlpha = 0.43;
                ctx.beginPath();
                ctx.arc(x, y, 6, 0, Math.PI * 2);
                ctx.stroke();
                ctx.restore();
            }
        }
        ctx.strokeStyle = strokeColor;
        ctx.lineWidth = 2.7 + 0.8 * Math.sin(arm + t/2);
        ctx.shadowBlur = 10 + 14 * Math.abs(Math.cos(t + arm));
        ctx.shadowColor = strokeColor;
        ctx.stroke();
        ctx.shadowBlur = 0;
    }
}

// Evolving particle rings
function drawParticleRing(t) {
    const numParticles = 48;
    const ringRadius = 230 + 19 * Math.sin(t * 0.5);
    for (let i = 0; i < numParticles; i++) {
        const theta = (2 * Math.PI / numParticles) * i + t * 0.09 * (1 + Math.sin(i));
        const localR = ringRadius
            + 12 * Math.sin(i + t * 1.5)
            + 7 * Math.cos(i * 2.5 + t * 1.3);

        const x = CENTER.x + localR * Math.cos(theta);
        const y = CENTER.y + localR * Math.sin(theta);
        const cT = (t * 0.18 + i / numParticles) % 1;

        ctx.beginPath();
        ctx.arc(x, y, 3.1 + 1.7 * Math.sin(i + t * 0.77), 0, 2 * Math.PI);
        ctx.fillStyle = getRainbowColor(0.93 - cT, 0.95);
        ctx.shadowBlur = 12;
        ctx.shadowColor = getRainbowColor(cT, 0.51);
        ctx.fill();
        ctx.shadowBlur = 0;
    }
}

// Central rippling pulses
function drawCentralRipples(t) {
    for (let k = 0; k < 3; k++) {
        const baseR = 32 + 10 * k + 6 * Math.sin(t + k * 1.33);
        ctx.beginPath();
        ctx.arc(
            CENTER.x,
            CENTER.y,
            baseR + 9 * Math.abs(Math.sin(t * 1.5 + k)),
            0, 2 * Math.PI
        );
        ctx.strokeStyle = getRainbowColor((t * 0.11 + k * 0.2) % 1, 0.37 + 0.33 * Math.abs(Math.sin(t + k)));
        ctx.lineWidth = 4 - k;
        ctx.shadowBlur = 7 + 8 * Math.abs(Math.sin(t + k));
        ctx.shadowColor = ctx.strokeStyle;
        ctx.stroke();
        ctx.shadowBlur = 0;
    }
}

function animate() {
    fadeCanvas();
    drawSinusoidalMandala(time);
    drawParticleRing(time);
    drawCentralRipples(time);
    time += 0.025;
    requestAnimationFrame(animate);
}

// Fill initial background
ctx.fillStyle = "#0a0120";
ctx.fillRect(0, 0, W, H);

animate();

</script>
