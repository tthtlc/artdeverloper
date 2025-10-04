---
layout: fullscreen
title: "Psychedelic Spiraling Waves"
tags:
  - graphics
---

<canvas id="spiralCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('spiralCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width;
const H = canvas.height;
const CX = W / 2;
const CY = H / 2;

// Dynamic color palette for psychedelic effects
function palette(t) {
    // t in [0,1]
    const r = Math.floor(200 + 55 * Math.sin(2 * Math.PI * (t + 0.2)));
    const g = Math.floor(200 + 55 * Math.sin(2 * Math.PI * (t + 0.5)));
    const b = Math.floor(200 + 55 * Math.sin(2 * Math.PI * (t + 0.8)));
    return `rgb(${r},${g},${b})`;
}

// Parameters for the spiral wave system
const waveCount = 17;
const waveDetail = 130;
const baseRadius = 95;
const spiralGrowth = 7; // radial step per wave
const timeAmpl = 28;
const arms = 9; // Number of spiral arms
const innerTwist = 0.23; // Controls spiral tightness

let t = 0;

function draw() {
    ctx.globalAlpha = 0.15;
    ctx.fillStyle = "#0b0520";
    ctx.fillRect(0, 0, W, H);
    ctx.globalAlpha = 1;

    for (let w = 0; w < waveCount; w++) {
        ctx.beginPath();
        for (let s = 0; s <= waveDetail; s++) {
            const a = (2 * Math.PI * s) / waveDetail;
            // Spiral out
            const r0 = baseRadius + spiralGrowth * w + timeAmpl * Math.sin(2.5 * a + t*0.8 + w * 0.2);
            // Psychedelic wobble
            const r = r0 + 18 * Math.sin(arms * a + t + w * 0.25) + 16 * Math.cos(2 * a - t * 0.6 + w);
            // Spiral unwinding
            const theta = a + innerTwist * w + Math.sin(t * 0.35 + w);
            const x = CX + r * Math.cos(theta);
            const y = CY + r * Math.sin(theta);

            if (s === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        // Modulate color based on wave index and time for a swirling rainbow effect
        ctx.strokeStyle = palette(((w/waveCount) + (t*0.07)) % 1);
        ctx.save();
        ctx.shadowColor = ctx.strokeStyle;
        ctx.shadowBlur = 8 + 13 * Math.abs(Math.sin(t*0.5 + w));
        ctx.lineWidth = 1.6 + 1.1*Math.cos(t*0.7+w*0.3);
        ctx.stroke();
        ctx.restore();
    }
    t += 0.028;
    requestAnimationFrame(draw);
}

// Start with dark background
ctx.fillStyle = "#0b0520";
ctx.fillRect(0, 0, W, H);

// Respond to window resizing
window.addEventListener('resize', () => {
    // Preserve aspect ratio
    let size = Math.min(window.innerWidth, window.innerHeight, 900);
    canvas.width = canvas.height = size;
});

// Kick off animation
draw();
</script>
