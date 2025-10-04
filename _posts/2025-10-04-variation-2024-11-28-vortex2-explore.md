---
layout: fullscreen
title: "Radiant Spiro-Splash: Animated Waveforms & Particle Bursts"
tags:
  - graphics
---

<style>
    body {
        background: radial-gradient(circle, #18082c 0%, #00061c 100%);
        margin: 0;
    }
    canvas {
        display: block;
        margin: 32px auto;
        background: transparent;
        border-radius: 16px;
        box-shadow: 0 0 32px rgba(80,30,150,0.45);
    }
    .controls {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 8px;
        margin-bottom: 8px;
    }
    .control-group {
        display: flex;
        flex-direction: column;
        align-items: center;
        color: #f0e9fe;
        font-family: 'Fira Mono', monospace;
        font-size: 1.1em;
    }
    label {
        letter-spacing: 1px;
        margin-bottom: 2px;
    }
    input[type="range"] {
        width: 200px;
    }
    #canvas {
        background: transparent;
        border: 1.5px solid #5d39f7;
    }
</style>

<canvas id="canvas" width="850" height="850"></canvas>
<div class="controls">
    <div class="control-group">
        <label for="petals">Petals:</label>
        <input type="range" id="petals" min="3" max="20" step="1" value="7">
        <span id="value-petals">7</span>
    </div>
    <div class="control-group">
        <label for="waves">Waves:</label>
        <input type="range" id="waves" min="1" max="8" step="1" value="4">
        <span id="value-waves">4</span>
    </div>
    <div class="control-group">
        <label for="radius">Radius:</label>
        <input type="range" id="radius" min="150" max="390" step="1" value="290">
        <span id="value-radius">290</span>
    </div>
    <div class="control-group">
        <label for="burst">Burst:</label>
        <input type="range" id="burst" min="0" max="100" step="1" value="42">
        <span id="value-burst">42</span>
    </div>
    <div class="control-group">
        <label for="speed">Speed:</label>
        <input type="range" id="speed" min="10" max="100" step="1" value="30">
        <span id="value-speed">30</span>
    </div>
    <div class="control-group">
        <label for="particles">Particles:</label>
        <input type="range" id="particles" min="24" max="111" step="1" value="60">
        <span id="value-particles">60</span>
    </div>
</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;

// Controls, values, and user input binding
const controls = ['petals','waves','radius','burst','speed','particles'];
const params = {};
const valueSpans = {};
for (let c of controls) {
    params[c] = parseFloat(document.getElementById(c).value);
    valueSpans[c] = document.getElementById('value-' + c);
    document.getElementById(c).addEventListener('input', (e) => {
        params[c] = parseFloat(e.target.value);
        valueSpans[c].textContent = params[c];
    });
}

// Animate! Main loop variables
let t = 0;
const TWO_PI = Math.PI * 2;
const center = {x: W/2, y: H/2};

// Utility: HSL rainbow
function hsl(h,s=80,l=65,a=1) {
    return `hsla(${(h)%360},${s}%,${l}%,${a})`;
}

// Utility: easeInOutSine for smooth transitions
function easeSine(x) {
    return -(Math.cos(Math.PI * x) - 1)/2;
}

function drawWaveFlower(time) {
    const {petals, waves, radius, burst, speed} = params;
    ctx.save();
    ctx.translate(center.x, center.y);
    ctx.globalCompositeOperation = "lighter";
    const R = radius * (0.93 + 0.07 * Math.sin(time*0.27));
    const burstF = burst/100;

    for(let l=0; l<2; l++) { // two layers, offset
        ctx.beginPath();
        for(let i=0;i<=TWO_PI+0.01;++i){
            // animated wave modulation
            const angle = i;
            const wave = Math.sin(petals*angle + l*Math.PI/petals + time*0.7) 
               + 0.29 * Math.cos(waves*angle - time*1.0 - l*1.8);

            const r = R * (1 + burstF * Math.sin(waves*angle+time*1.3 + l*0.5)*easeSine(Math.sin(time*0.5+angle)));
            const x = Math.cos(angle) * (r + 33*wave);
            const y = Math.sin(angle) * (r + 33*wave);
            ctx.lineTo(x,y);
        }
        const hue = (time*22 + l*144)%360;
        ctx.strokeStyle = hsl(hue,92,60,0.89);
        ctx.shadowColor = hsl(hue+40,80,67,0.39);
        ctx.shadowBlur = 36 + Math.abs(Math.sin(time))*30;
        ctx.lineWidth = 7-l*3.5;
        ctx.stroke();
    }
    ctx.restore();
}

function drawRadiantParticles(time) {
    const {particles, radius, speed, burst, petals, waves} = params;
    ctx.save();
    ctx.translate(center.x, center.y);
    const baseR = radius * (0.91 + 0.09 * Math.sin(time*0.37));
    for(let i=0;i<particles;i++) {
        const a = i*TWO_PI/particles;
        const waveR = 1 + 0.13 * Math.sin(petals*a + waves*Math.cos(a*burst/70 + time*1.02));
        const r = baseR*waveR + 32*Math.abs(Math.sin(time + a*waves/2));
        const px = Math.cos(a) * r;
        const py = Math.sin(a) * r;
        const cyc = 0.7 + 0.3*Math.sin(time + a*waves);
        const hue = (a*160/TWO_PI + Math.sin(time*speed/170+a*4.0)*90 + time*23) % 360;
        ctx.beginPath();
        ctx.arc(px, py, 8 + 7*Math.abs(cyc), 0, TWO_PI);
        ctx.fillStyle = hsl(hue, 95, 50+17*Math.sin(a*petals+time*0.5), 0.16+0.13*cyc);
        ctx.globalAlpha = 0.61 + 0.3*cyc;
        ctx.shadowColor = hsl(hue+30, 90, 80, 0.45);
        ctx.shadowBlur = 22 + 10*Math.abs(Math.sin(a*3+time));
        ctx.fill();
        ctx.globalAlpha = 1.0;
    }
    ctx.restore();
}

function drawPsyPulse(time) {
    // Summon a set of rippling psy "halos"
    ctx.save();
    ctx.translate(center.x, center.y);
    for(let h=0;h<4;++h){
        const baseR = 380 + 25*Math.sin(time*0.8+h*1.5);
        ctx.beginPath();
        for(let t=0; t<=TWO_PI+0.01; t+=0.024){
            const off = 13*Math.sin(params.petal * t + h + time*0.7) + 
                        9*Math.sin(params.waves * t + time*1.08 - h*1.2);
            ctx.lineTo(Math.cos(t)*(baseR+off), Math.sin(t)*(baseR+off));
        }
        const hue = (120*h + time*25)%360;
        ctx.strokeStyle = hsl(hue, 80, 60, 0.13);
        ctx.lineWidth = 2 + 1.5*Math.sin(h + time*1.1);
        ctx.shadowColor = hsl(hue,92,70,0.30);
        ctx.shadowBlur = 11;
        ctx.stroke();
    }
    ctx.restore();
}


// Animate
function animate() {
    t += 0.014 + params.speed/2500;
    ctx.clearRect(0,0,W,H);
    // Draw outer psychedelic pulses
    drawPsyPulse(t);
    // Draw the main flower/waveform shape
    drawWaveFlower(t);
    // Particle bursts/orbs
    drawRadiantParticles(t);
    requestAnimationFrame(animate);
}
animate();

</script>
