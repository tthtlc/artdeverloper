```markdown
---
layout: fullscreen
title: "Psychedelic Ripple Waves with Pulsating Circles"
tags:
  - graphics
---

<canvas id="rippleCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('rippleCanvas');
const ctx = canvas.getContext('2d');

// Configuration
const W = canvas.width, H = canvas.height;
const CENTER = { x: W/2, y: H/2 };
const NUM_WAVES = 9;
const RING_STEP = 34;
const MAX_R = Math.min(W,H)/2-16;
const NUM_CIRCLES = 36;

let time = 0;

// Psychedelic color palette generator (based on sine functions)
function rainbowColor(t, offset=0) {
    const f = (n) => Math.round(127.5 + 127.5 * Math.sin(t + offset + n));
    return `rgb(${f(0)},${f(2)},${f(4)})`;
}

// Draw a wavy circle (ripple)
function drawRipple(cx, cy, baseR, waveAmp, waveFreq, phase, color) {
    ctx.save();
    ctx.strokeStyle = color;
    ctx.lineWidth = 3 + 2*Math.sin(phase); // pulse a bit!
    ctx.beginPath();
    for (let a=0; a<=Math.PI*2 + 0.01; a += Math.PI/60) {
        // Rippled radius
        let r = baseR + waveAmp * Math.sin(waveFreq*a + phase*0.8);
        let x = cx + r * Math.cos(a);
        let y = cy + r * Math.sin(a);
        if (a===0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
    }
    ctx.closePath();
    ctx.shadowColor = color;
    ctx.shadowBlur = 16 + 10*Math.sin(phase);
    ctx.stroke();
    ctx.restore();
}

// Draw floating, pulsing circles along big circle path
function drawOrbs(cx, cy, orbitR, count, t) {
    for (let i=0; i<count; ++i) {
        let angle = (2*Math.PI/count)*i + t*0.75 + Math.sin(t/4+i)*0.5;
        let subR = 19 + 14 * Math.sin(t + angle*3 + i); // pulsate
        let ex = cx + (orbitR + 14*Math.sin(t*1.2 + i)) * Math.cos(angle);
        let ey = cy + (orbitR + 14*Math.sin(t*1.2 + i)) * Math.sin(angle);

        // Cool color for orb
        let col = rainbowColor(t*1.1 + i*0.9, i*1.15);
        ctx.save();
        ctx.beginPath();
        ctx.arc(ex, ey, subR, 0, 2*Math.PI);
        ctx.globalAlpha = 0.39 + 0.33 * Math.sin(t+i);
        ctx.shadowColor = col;
        ctx.shadowBlur = 22;
        ctx.fillStyle = col;
        ctx.fill();
        ctx.globalAlpha = 1;
        ctx.lineWidth = 2;
        ctx.strokeStyle = "#FFF";
        ctx.stroke();
        ctx.restore();
    }
}

// Main animation loop
function draw() {
    ctx.clearRect(0,0,W,H);

    // Ripple waves
    for (let i=NUM_WAVES-1; i>=0; --i) {
        let baseR = MAX_R - i * RING_STEP;
        let amp = 15 + 6 * Math.sin(time*0.88 + i);
        let freq = 3.7 + 2 * Math.sin(time*.26 + i*0.95);
        let c = rainbowColor(time + i*Math.PI/7, i);
        let phase = time*0.75 + i*0.7;
        drawRipple(CENTER.x, CENTER.y, baseR, amp, freq, phase, c);
    }

    // Pulsating orbiting circles
    drawOrbs(CENTER.x, CENTER.y, MAX_R - RING_STEP/2 - 7, NUM_CIRCLES, time);

    // Central glowing "sun"
    ctx.save();
    let grad = ctx.createRadialGradient(CENTER.x,CENTER.y,2,CENTER.x,CENTER.y,44);
    grad.addColorStop(0, '#FFF99F');
    grad.addColorStop(0.45, rainbowColor(time*0.82,0));
    grad.addColorStop(1, '#222200');
    ctx.beginPath();
    ctx.arc(CENTER.x,CENTER.y,43 + 11*Math.sin(time),0,Math.PI*2);
    ctx.fillStyle = grad;
    ctx.shadowColor = rainbowColor(time*0.7);
    ctx.shadowBlur = 25;
    ctx.fill();
    ctx.restore();

    time += 0.03;
    requestAnimationFrame(draw);
}

draw();
</script>
```
