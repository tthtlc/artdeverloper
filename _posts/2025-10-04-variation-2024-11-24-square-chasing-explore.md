---
layout: fullscreen
title: "Psychedelic Rotating Waveform Orbs"
tags:
  - graphics
---

<style>
canvas {
    background: radial-gradient(ellipse at center, #231942 0%, #5E548E 100%);
    display: block;
    margin: 30px auto 0 auto;
    border-radius: 12px;
    box-shadow: 0 3px 24px rgba(40,0,70,0.25);
}
.slider-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 10px;
}
label {
    margin-bottom: 5px;
    color: #fff;
    font-size: 1.1em;
    letter-spacing: 1px;
    font-family: sans-serif;
}
input[type="range"] {
    width: 350px;
}
</style>
<div class="slider-container">
  <label for="waveSlider">Wave Complexity</label>
  <input type="range" id="waveSlider" min="2" max="12" value="5" step="1">
</div>
<canvas id="canvas" width="700" height="700"></canvas>
<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const waveSlider = document.getElementById('waveSlider');

let W = canvas.width;
let H = canvas.height;

let waveComplexity = parseInt(waveSlider.value);

function orbs(t) {
    // Orb settings
    const count = 7; // number of orbs
    const baseRadii = [180, 110, 75, 50, 30, 20, 12];
    for (let o = 0; o < count; o++) {
        const r = baseRadii[o];
        const cx = W/2;
        const cy = H/2;
        // Each orb will rotate
        const phi = t * (0.04 + 0.008 * o) + o * Math.PI/4;
        ctx.save();
        ctx.translate(cx, cy);
        ctx.rotate(phi);
        // Orbs will "breathe"
        const scale =
            1 + 0.10 * Math.sin(t * (0.45+0.07*o) + o) +
            0.02 * Math.sin(t * (0.6+0.13*o) + 2*o);
        ctx.scale(scale, scale);
        drawWobblyOrb(r, o, t);
        ctx.restore();
    }
}

function drawWobblyOrb(R, orbIndex, time) {
    const nPoints = 180;
    const w = waveComplexity + orbIndex;
    ctx.beginPath();
    // Random phase offset per orb for the waviness
    const phi_off = orbIndex * Math.PI/6;
    for (let i = 0; i <= nPoints; i++) {
        const a = i * 2*Math.PI / nPoints;
        // Amplitude of wave
        const waveAmp = 13 + 9*Math.sin(time*0.33 + orbIndex+a*2);
        // Modulate radius with a multi-harmonic wave
        const r =
            R +
            waveAmp*Math.sin(w*a + phi_off + time*0.6 + orbIndex) +
            5*Math.sin(a*waveComplexity*1.5 - time*0.8 + orbIndex*2);

        const x = Math.cos(a) * r;
        const y = Math.sin(a) * r;
        if(i===0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
    }

    // Colorful gradient stroke
    // We'll gradient along the stroke by segment drawing
    for (let i = 0; i < nPoints; i++) {
        const a1 = i * 2*Math.PI / nPoints;
        const a2 = (i+1) * 2*Math.PI / nPoints;
        // Wavy radius
        const waveAmp = 13 + 9*Math.sin(time*0.33 + orbIndex+a1*2);
        const r1 =
            R +
            waveAmp*Math.sin(w*a1 + phi_off + time*0.6 + orbIndex) +
            5*Math.sin(a1*waveComplexity*1.5 - time*0.8 + orbIndex*2);
        const x1 = Math.cos(a1)*r1;
        const y1 = Math.sin(a1)*r1;

        const waveAmp2 = 13 + 9*Math.sin(time*0.33 + orbIndex+a2*2);
        const r2 =
            R +
            waveAmp2*Math.sin(w*a2 + phi_off + time*0.6 + orbIndex) +
            5*Math.sin(a2*waveComplexity*1.5 - time*0.8 + orbIndex*2);
        const x2 = Math.cos(a2)*r2;
        const y2 = Math.sin(a2)*r2;

        // Color: psychedelic rainbow waves, each orb offset
        const hue =
            ((time*14 + a1*180/Math.PI + orbIndex*49) % 360 + 360) % 360;
        const sat = 83 + 10*Math.sin(orbIndex*2 + time*0.6 + a1*3);
        const light = 54 + 18*Math.sin(orbIndex + time*0.3 + a1*6);

        ctx.strokeStyle = `hsl(${hue},${sat}%,${light}%)`;
        ctx.lineWidth = 3.5 - orbIndex*0.4;
        ctx.beginPath();
        ctx.moveTo(x1,y1);
        ctx.lineTo(x2,y2);
        ctx.stroke();
    }
}

// Animated trippy background grid
function psychedelicGrid(time) {
    const waves = 21;
    const step = W/(waves-1);
    for(let i=0; i<waves; i++) {
        for(let j=0; j<waves; j++) {
            const x = i*step;
            const y = j*step;
            // Oscillating offset
            const dx = 15 * Math.sin( (i+j)/4 + time*0.9 + Math.sin(x*y*1e-5+time*0.3)*1.7 );
            const dy = 15 * Math.cos( (i-j)/5 - time*1.4 + Math.cos(y*x*1e-5+time*0.6)*2.2 );
            // Colorful backdrop dots
            const hue = (180 + 80*Math.sin(time*0.23 + i) + 120*Math.sin(time*0.18 + j)) % 360;
            ctx.beginPath();
            ctx.arc(x+dx, y+dy, 2 + 2*Math.abs(Math.sin(dx*0.4 + dy*0.2)), 0, 2*Math.PI);
            ctx.fillStyle = `hsla(${hue}, 53%, 19%, 0.32)`;
            ctx.fill();
        }
    }
}


function draw(now) {
    const t = now*0.001; // seconds

    // Draw dreamy background
    ctx.clearRect(0, 0, W, H);

    // Trippy grid
    psychedelicGrid(t);

    // Orbital morphing rainbow shapes
    orbs(t);

    // Inner, central glowing point
    let glowGradient = ctx.createRadialGradient(W/2, H/2, 0, W/2, H/2, 90);
    glowGradient.addColorStop(0, 'rgba(255,255,220,0.27)');
    glowGradient.addColorStop(1, 'rgba(120,0,160,0)');
    ctx.beginPath();
    ctx.arc(W/2, H/2, 90, 0, 2*Math.PI);
    ctx.fillStyle = glowGradient;
    ctx.fill();

    requestAnimationFrame(draw);
}

requestAnimationFrame(draw);

waveSlider.addEventListener('input', () => {
    waveComplexity = parseInt(waveSlider.value);
});
</script>
