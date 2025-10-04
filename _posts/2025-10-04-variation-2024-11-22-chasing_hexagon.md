---
layout: fullscreen
title: "Shimmering Chromatic Spirals"
tags:
  - graphics
---

<canvas id="psySpiralCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById("psySpiralCanvas");
const ctx = canvas.getContext("2d");

// Canvas size and center
const W = canvas.width;
const H = canvas.height;
const CX = W/2;
const CY = H/2;

// Settings for spiral arms and particles
const ARMS = 8;
const PARTICLES_PER_ARM = 140;
const SPIRAL_RADIUS = 220;
const PARTICLE_SIZE = 8;

// For coloring and animation
let t = 0;

// Helpers for oscillation
function oscillate(x, amp, freq, phase=0) {
  return amp * Math.sin(freq * x + phase);
}

// The core render loop
function draw() {
    ctx.globalAlpha = 0.12;
    ctx.fillStyle = "#12031a";  // fading dark background for persistence
    ctx.fillRect(0,0,W,H);
    ctx.globalAlpha = 1;
    
    for (let arm = 0; arm < ARMS; arm++) {
        let armPhase = (2*Math.PI/ARMS) * arm + t*0.6 + Math.sin(t*0.18 + arm)*0.6;
        for (let i = 0; i < PARTICLES_PER_ARM; i++) {
            // Spiral equation: r = a + b * theta
            let theta = i*0.17 + armPhase;
            let radius = SPIRAL_RADIUS * (i/(PARTICLES_PER_ARM*1.1)) + oscillate(i,14,0.09,arm*1.5+ t*0.5);
            
            // Oscillate the spiral a bit
            let wave = oscillate(i+t*1.3, 31, 0.11, arm*0.29);

            let x = CX + (radius + wave) * Math.cos(theta);
            let y = CY + (radius + wave) * Math.sin(theta);

            // Psychedelic coloring (HSLA rainbow, modulated over t and i)
            let hue = (arm*45 + i*2.5 + t*17)%360;
            let sat = 65 + 30*Math.sin(arm + t*0.7 + i*0.13);
            let light = 59 + 16*Math.sin(i*0.08 + t*0.9 - arm*0.4);

            ctx.save();

            // Shimmer with shadow and blur for glow
            ctx.beginPath();
            ctx.arc(x, y, PARTICLE_SIZE - 3*Math.sin(i*0.17 + t + arm), 0, 2*Math.PI);
            ctx.shadowColor = `hsla(${hue},95%,65%,0.85)`;
            ctx.shadowBlur = 22 + 12*Math.sin(t*1.1 + i*0.17 + arm);
            ctx.fillStyle = `hsla(${hue},${sat}%,${light}%,0.74)`;
            ctx.globalCompositeOperation = 'lighter';
            ctx.fill();
            ctx.globalCompositeOperation = 'source-over';
            ctx.restore();
        }
    }
    t += 0.012;
    requestAnimationFrame(draw);
}

draw();
</script>
