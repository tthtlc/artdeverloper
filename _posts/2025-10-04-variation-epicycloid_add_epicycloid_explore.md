---
layout: fullscreen
title: Psychedelic Lissajous Waveform Spirals with Feedback Echoes
tags:
  - graphics
---

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Psychedelic Lissajous Waveform Spirals with Feedback Echoes</title>
<style>
    html, body {
        width: 100vw;
        height: 100vh;
        margin: 0;
        padding: 0;
        background: #161515;
        overflow: hidden;
        font-family: monospace;
    }
    body {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }
    .controls {
        position: fixed;
        left: 0;
        right: 0;
        bottom: 30px;
        display: flex;
        justify-content: center;
        gap: 40px;
        z-index: 2;
        background: rgba(12,12,12,0.7);
        border-radius: 2em;
        padding: 12px 20px 12px 20px;
        box-shadow: 0 6px 16px rgba(0,0,0,0.15);
        user-select: none;
    }
    .control-group {
        display: flex;
        align-items: center;
        gap: 6px;
    }
    .value-label {
        width: 28px;
        color: #cdf;
        display: inline-block;
        font-weight: bold;
        text-align: right;
    }
    input[type="range"] {
        accent-color: #7be;
    }
    .color-spectrum {
        width: 120px;
        height: 18px;
        border-radius: 8px;
        border: 1px solid #333;
        vertical-align: middle;
        margin-right: 8px;
    }
    #saveButton {
        background: #2780ff;
        color: #fff;
        border: none;
        border-radius: 0.6em;
        font-size: 1em;
        padding: 7px 16px;
        cursor: pointer;
        margin-left: 8px;
        box-shadow: 0 2px 8px #0005;
        transition: background .13s;
    }
    #saveButton:hover { background: #5aaaff }
</style>
</head>
<body>
<canvas id="canvas" width="900" height="900"></canvas>
<div class="controls">

    <div class="control-group">
        <label for="freqA">Freq X:</label>
        <input type="range" id="freqA" min="1" max="11" value="3">
        <span id="freqA-value" class="value-label">3</span>
    </div>
    <div class="control-group">
        <label for="freqB">Freq Y:</label>
        <input type="range" id="freqB" min="1" max="11" value="4">
        <span id="freqB-value" class="value-label">4</span>
    </div>
    <div class="control-group">
        <label for="rotVel">Rot Velocity:</label>
        <input type="range" id="rotVel" min="-20" max="20" value="8">
        <span id="rotVel-value" class="value-label">8</span>
    </div>
    <div class="control-group">
        <label for="size">Size:</label>
        <input type="range" id="size" min="180" max="440" value="320">
        <span id="size-value" class="value-label">320</span>
    </div>
    <div class="control-group">
        <label for="hue">Base Hue:</label>
        <canvas id="gradientCanvas" class="color-spectrum"></canvas>
        <input type="range" id="hue" min="0" max="360" value="220">
        <span id="hue-value" class="value-label">220</span>
    </div>
    <button id="saveButton">Save as PNG</button>
</div>
<script>
function clamp(x,a,b){return Math.max(a,Math.min(b,x));}

// -------- Controls/Parameters ----------
let freqA = 3;
let freqB = 4;
let rotVel = 8;
let patternSize = 320;
let baseHue = 220;

const $freqA = document.getElementById('freqA');
const $freqB = document.getElementById('freqB');
const $rotVel = document.getElementById('rotVel');
const $size = document.getElementById('size');
const $hue = document.getElementById('hue');

function updateControls() {
    document.getElementById('freqA-value').innerText = freqA;
    document.getElementById('freqB-value').innerText = freqB;
    document.getElementById('rotVel-value').innerText = rotVel;
    document.getElementById('size-value').innerText = patternSize;
    document.getElementById('hue-value').innerText = baseHue;
    drawColorGradient();
}
$freqA.oninput = function(){ freqA = parseInt(this.value); updateControls(); };
$freqB.oninput = function(){ freqB = parseInt(this.value); updateControls(); };
$rotVel.oninput = function(){ rotVel = parseInt(this.value); updateControls(); };
$size.oninput  = function(){ patternSize = parseInt(this.value); updateControls(); };
$hue.oninput   = function(){ baseHue = parseInt(this.value); updateControls(); };

// ----------- Gradient preview -------------
const gradientCanvas = document.getElementById('gradientCanvas');
const gctx = gradientCanvas.getContext('2d');
function drawColorGradient() {
    const w = gradientCanvas.width, h = gradientCanvas.height;
    let grad = gctx.createLinearGradient(0,0,w,0);
    for(let i=0;i<=w;i+=1){
        let hue = (baseHue+i/(w)*150)%360;
        grad.addColorStop(i/(w),`hsl(${hue},90%,70%)`);
    }
    gctx.fillStyle=grad;
    gctx.fillRect(0,0,w,h);
}
drawColorGradient();

// -------------- Canvas + Feedback ---------------
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

let WIDTH = canvas.width, HEIGHT = canvas.height;
function resizeCanvas() {
    let s = Math.min(window.innerWidth, window.innerHeight) * 0.98;
    canvas.width = canvas.height = Math.round(s);
    WIDTH = canvas.width; HEIGHT = canvas.height;
}
window.addEventListener('resize',()=>{
    resizeCanvas();
});
resizeCanvas();

// -------- Psychedelic Waveform Spiral Animation -------
let time = 0;
function drawLissajousSpiral(ctx, t, params) {
    const {
        freqA, freqB, rot, spiralArmCount, size, hueBase
    } = params;
    let cx = WIDTH/2, cy = HEIGHT/2;
    let totalPoints = 2200;
    let spiralRevolutions = 3.4;
    let armFadeExponent = 1.7;
    for(let arm=0; arm<spiralArmCount; ++arm){
        let phiOffset = (2*Math.PI/spiralArmCount)*arm + t*0.053*arm;
        ctx.beginPath();
        for(let i=0; i<=totalPoints; ++i){
            let p = i/totalPoints;
            let theta = spiralRevolutions*2*Math.PI*p + phiOffset;

            // Lissajous core with spiral radial modulation:
            let waveX = Math.sin(theta*freqA+t*0.36+arm*1.5);
            let waveY = Math.cos(theta*freqB-t*0.43-arm);

            let spiralR = size * p * (1.08 + 0.22 * Math.sin(p*8 + t*0.9 + arm));
            let jitter = 8*Math.sin(arm*2 + t + theta*3 + Math.sin(theta+t*0.4));
            let x = cx + (waveX*0.6+waveY*0.4) * spiralR * Math.sin(p*Math.PI/2) + spiralR*Math.cos(theta)*0.15 + jitter;
            let y = cy + (waveY*0.5+waveX*0.3) * spiralR * Math.sin(p*Math.PI/2) + spiralR*Math.sin(theta)*0.15 + jitter;

            if(i===0) ctx.moveTo(x,y); else ctx.lineTo(x,y);
        }
        let armFade = Math.pow(0.65+0.35*Math.cos(phiOffset+t*0.23),armFadeExponent);
        let hue = (hueBase+arm*360/spiralArmCount+t*6)%360;
        let sat = 80-24*armFade;
        let light = 65+15*Math.abs(Math.sin(phiOffset*1.3+t*0.2));
        ctx.strokeStyle = `hsla(${hue},${sat}%,${light}%,${0.7*armFade})`;
        ctx.shadowColor = `hsl(${hue},98%,78%)`;
        ctx.shadowBlur = 16+8*armFade;
        ctx.lineWidth = 2+armFade*3.8;
        ctx.stroke();
    }
}

// Feedback color fade
function feedbackEcho(ctx, alpha=0.14) {
    ctx.save();
    ctx.globalAlpha = alpha;
    ctx.globalCompositeOperation = 'lighter';
    ctx.fillStyle = '#181917';
    ctx.fillRect(0,0,WIDTH,HEIGHT);
    ctx.restore();
}

function animate() {
    // Feedback echo - leaves fading "afterimages"
    feedbackEcho(ctx, 0.11);

    // Draw the spiral pattern
    let spiralArms = 5 + Math.floor(Math.abs(Math.sin(time*0.24))*5);
    drawLissajousSpiral(ctx, time, {
        freqA, freqB, rot:time*0.07, spiralArmCount:spiralArms,
        size:patternSize,
        hueBase:baseHue
    });

    time += rotVel*0.0034;
    requestAnimationFrame(animate);
}
animate();

// Save PNG
document.getElementById('saveButton').onclick = function(){
    let timestamp = new Date().toISOString().replace(/[:.]/g,'');
    let fname = `psy_lissajous_${freqA}x${freqB}_h${baseHue}_${timestamp}.png`;
    let link = document.createElement('a');
    link.download = fname;
    link.href = canvas.toDataURL('image/png');
    link.click();
};

// Prevent context menu
document.addEventListener("contextmenu", ev=>ev.preventDefault());
</script>
</body>
</html>
```
