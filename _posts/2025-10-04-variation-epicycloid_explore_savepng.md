---
layout: fullscreen
title: Psychedelic Harmonic Wave Fields with Color Trails
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Psychedelic Harmonic Wave Fields with Color Trails</title>
    <style>
        body {
            margin: 0;
            background: #000;
            display: flex;
            flex-direction: column;
            align-items: center;
            height: 100vh;
            justify-content: center;
        }
        canvas {
            border: 1px solid #222;
            background: #000;
            display: block;
        }
        .controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-top: 20px;
            color: #eee;
            font-family: monospace;
            user-select: none;
        }
        .control-group {
            margin: 5px 0;
            display: flex;
            align-items: center;
        }
        .control-group label {
            margin-right: 8px;
        }
        .value-label {
            margin-left: 10px;
            font-weight: bold;
            min-width: 2em;
            color: #b3ff67;
        }
        input[type="range"] {
            width: 180px;
        }
        .color-spectrum {
            margin: 10px 0 10px 0;
            width: 250px;
            height: 30px;
            border-radius: 8px;
            box-shadow: 0 0 10px #444;
        }
        #saveButton {
            margin-top: 12px;
            background: #181830;
            color: #f7c23e;
            border: none;
            padding: 7px 18px;
            border-radius: 5px;
            font-family: monospace;
            font-weight: bold;
            font-size: 1em;
            cursor: pointer;
            box-shadow: 0 0 7px #181830;
        }
        #saveButton:hover {
            background: #f7c23e;
            color: #181830;
        }
    </style>
</head>
<body>
<canvas id="canvas" width="720" height="600"></canvas>

<div class="controls">
    <div class="control-group">
        <label for="waves"># Harmonic Waves:</label>
        <input type="range" id="waves" min="2" max="8" value="4">
        <span class="value-label" id="waves-value">4</span>
    </div>
    <div class="control-group">
        <label for="freq">Base Frequency:</label>
        <input type="range" id="freq" min="20" max="140" value="50">
        <span class="value-label" id="freq-value">50</span>
    </div>
    <div class="control-group">
        <label for="speed">Speed:</label>
        <input type="range" id="speed" min="2" max="20" value="6">
        <span class="value-label" id="speed-value">6</span>
    </div>
    <div class="control-group">
        <label for="trails">Trail Length:</label>
        <input type="range" id="trails" min="10" max="110" value="50">
        <span class="value-label" id="trails-value">50</span>
    </div>
    <div class="control-group">
        <label for="phaseShift">Phase Warp:</label>
        <input type="range" id="phaseShift" min="0" max="314" value="157">
        <span class="value-label" id="phaseShift-value">1.57</span>
    </div>
    <div class="control-group">
        <label for="color">Palette Start:</label>
        <input type="color" id="color" value="#31eaff">
    </div>
    <canvas id="gradientCanvas" width="250" height="30" class="color-spectrum"></canvas>
    <button id="saveButton">Save as PNG</button>
</div>

<script>
document.addEventListener("contextmenu", e => e.preventDefault());

const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const gradientCanvas = document.getElementById('gradientCanvas');
const gradientCtx = gradientCanvas.getContext('2d');

let waveCount = 4;
let baseFreq = 50;
let speed = 6;
let trailLength = 50;
let basePhaseShift = Math.PI/2;
let selectedColor = "#31eaff";
let lastFrame = 0;
let w = canvas.width, h = canvas.height;

function lerp(a, b, t) { return a + (b - a) * t; }
function clamp(v, min, max) { return Math.max(min, Math.min(v, max)); }

// Convert HSV to RGB for rainbow trails
function hsv2rgb(h, s, v) {
    let f = (n, k = (n + h/60)%6) => v - v*s*Math.max(Math.min(k,4-k,1),0);
    return [f(5),f(3),f(1)];
}
// So controls show initial values:
["waves","freq","speed","trails","phaseShift"].forEach(id=>{
    const slider = document.getElementById(id);
    const valLbl = document.getElementById(id+'-value');
    let v = slider.value;
    if(id=="phaseShift") v = (v/100).toFixed(2);
    valLbl.innerText = v;
})

function drawGradientBar() {
    const steps = 32;
    gradientCtx.clearRect(0, 0, gradientCanvas.width, gradientCanvas.height);
    for(let i=0;i<steps;i++){
        let t = i/(steps-1);
        let h = (t*360 + baseColorH() ) % 360;
        let col = hsv2rgb(h, 0.6, 1.0);
        gradientCtx.fillStyle = `rgb(${col.map(v=>Math.round(v*255)).join(',')})`;
        gradientCtx.fillRect(t*gradientCanvas.width, 0, gradientCanvas.width/steps+2, gradientCanvas.height);
    }
}
function baseColorH() {
    // base color as hue
    let rgb = hexToRgb(selectedColor);
    let mx = Math.max(rgb.r, rgb.g, rgb.b), mn = Math.min(rgb.r, rgb.g, rgb.b);
    let h,s,v;
    v = mx/255;
    s = mx==0? 0: (mx-mn)/mx;
    if(mx==mn)h=0;
    else if(mx==rgb.r) h=(60*((rgb.g - rgb.b)/(mx-mn))+360)%360;
    else if(mx==rgb.g) h=(60*((rgb.b - rgb.r)/(mx-mn))+120)%360;
    else h=(60*((rgb.r - rgb.g)/(mx-mn))+240)%360;
    return h;
}
function hexToRgb(hex) {
    let bigint = parseInt(hex.slice(1), 16);
    let r = (bigint >> 16) & 255,
        g = (bigint >> 8) & 255,
        b = bigint & 255;
    return { r, g, b };
}

// Harmonic field generator
function drawHarmonicWaveField(time) {
    // Fade old trails
    ctx.globalAlpha = 1.0;
    ctx.fillStyle = "rgba(0,0,0," + (1.15 - clamp(trailLength/110,0.01,0.7)) + ")";
    ctx.fillRect(0,0,w,h);

    let cols = 140, rows = 70, amp = h/3.2;
    let marginX = w*0.08, marginY = h*0.1;
    for(let c=0; c<cols; c++) {
        let x = lerp(marginX, w-marginX, c/(cols-1));
        for(let r=0; r<rows; r++) {
            let y = lerp(marginY, h-marginY, r/(rows-1));
            // Build the value by summing several transverse sine waves with different freq/phase
            let out = 0, hue = 0;
            for(let j=0;j<waveCount;j++) {
                let freq = baseFreq/30 + 0.14*j;
                let phase = time*0.001*speed + j*basePhaseShift + Math.cos((x+y+j)*0.0012)*1.3*j;
                out += Math.sin(
                            freq*x*0.007+freq*y*0.007 + phase + Math.sin(time*0.002+j)*1.1
                        ) * (amp/(0.89+j));
                hue += 16+j*31+Math.cos(phase)*87;
            }
            out /= waveCount;
            hue = (hue/waveCount + baseColorH()) % 360;
            let px = x + out*Math.cos(time*0.0006+out*0.0002);
            let py = y + out*Math.sin(time*0.0005+out*0.0003);
            // Color cycling through HSV palettes (psychedelic look)
            let rgb = hsv2rgb(hue, 0.72, 0.96 - 0.26*Math.cos(time*0.001+hue*0.02));
            ctx.globalAlpha = 0.61;
            ctx.beginPath();
            ctx.arc(px, py, 1.5 + 1.2*Math.sin(time*0.004+hue), 0, 2*Math.PI);
            ctx.fillStyle = `rgb(${rgb.map(v=>Math.floor(v*255)).join(",")})`;
            ctx.fill();
        }
    }
    ctx.globalAlpha = 1.0;
}

function loop(now) {
    drawHarmonicWaveField(now || 0);
    requestAnimationFrame(loop);
}

// Control bindings
document.getElementById('waves').addEventListener('input', e=>{
    waveCount = parseInt(e.target.value);
    document.getElementById('waves-value').innerText = e.target.value;
});
document.getElementById('freq').addEventListener('input', e=>{
    baseFreq = parseInt(e.target.value);
    document.getElementById('freq-value').innerText = e.target.value;
});
document.getElementById('speed').addEventListener('input', e=>{
    speed = parseInt(e.target.value);
    document.getElementById('speed-value').innerText = e.target.value;
});
document.getElementById('trails').addEventListener('input', e=>{
    trailLength = parseInt(e.target.value);
    document.getElementById('trails-value').innerText = e.target.value;
});
document.getElementById('phaseShift').addEventListener('input', e=>{
    basePhaseShift = e.target.value/100;
    document.getElementById('phaseShift-value').innerText = (e.target.value/100).toFixed(2);
});
document.getElementById('color').addEventListener('input', e=>{
    selectedColor = e.target.value;
    drawGradientBar();
});
document.getElementById('saveButton').addEventListener('click', ()=>{
    const link = document.createElement('a');
    const filename = `harmonic-field_${waveCount}waves_f${baseFreq}_spd${speed}_t${trailLength}.png`;
    link.download = filename;
    link.href = canvas.toDataURL('image/png');
    link.click();
});

drawGradientBar();
requestAnimationFrame(loop);
</script>
</body>
</html>
```