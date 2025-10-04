---
layout: fullscreen
title: "Psychedelic Waveform Grid with Kinetic Lissajous Particles (Sliders, Palette, Save)"
tags:
  - graphics
---

<style>
.canvas-container {
    display: flex;
    flex-direction: column;
    align-items: center;
}
canvas {
    border: 1px solid black;
    margin-top: 12px;
}
.controls {
    margin-top: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    font-family: Arial, sans-serif;
    background: rgba(240,240,255,0.91);
    border-radius: 10px;
    padding: 20px 35px 18px 35px;
    box-shadow: 0 0 7px #e0e0ff;
}
.control-group {
    margin: 9px 0;
    display: flex;
    align-items: center;
}
.control-group label {
    min-width: 120px;
    margin-right: 8px;
}
input[type="range"] {
    width: 180px;
}
.value-label {
    min-width: 22px;
    text-align: right;
    margin-left: 12px;
    font-weight: bold;
    color: #555;
    font-family: monospace;
    font-size: 1.05em;
}
#paletteCanvas {
    width: 250px;
    height: 30px;
    margin-bottom: 7px;
    border: 1px solid #aaa;
    border-radius: 5px;
}
#saveButton {
    margin-top: 10px;
    padding: 5px 20px;
    background: #8888fa;
    color: white;
    font-size: 1.05em;
    border-radius: 6px;
    border: none;
    cursor: pointer;
}
#saveButton:hover {
    background: #5f54a8;
}
@media (max-width: 700px) {
  canvas { width: 97vw !important; height: 80vw !important; }
}
</style>
<div class="canvas-container">
    <canvas id="psycanvas" width="750" height="750"></canvas>
    <div class="controls">
        <div class="control-group">
            <label for="cellsize">Cell Size:</label>
            <input type="range" id="cellsize" min="48" max="160" value="80">
            <span id="cellsize-value" class="value-label">80</span>
        </div>
        <div class="control-group">
            <label for="freqx">Freq X:</label>
            <input type="range" id="freqx" min="1" max="18" value="5">
            <span id="freqx-value" class="value-label">5</span>
        </div>
        <div class="control-group">
            <label for="freqy">Freq Y:</label>
            <input type="range" id="freqy" min="1" max="18" value="8">
            <span id="freqy-value" class="value-label">8</span>
        </div>
        <div class="control-group">
            <label for="particles">Particles:</label>
            <input type="range" id="particles" min="2" max="12" value="8">
            <span id="particles-value" class="value-label">8</span>
        </div>
        <div class="control-group">
            <label for="trail">Trail Fade:</label>
            <input type="range" id="trail" min="0" max="99" value="45">
            <span id="trail-value" class="value-label">45</span>
        </div>
        <div class="control-group">
            <label for="palette">Base Palette:</label>
            <input type="color" id="palette" value="#ff4eb6">
        </div>
        <canvas id="paletteCanvas" width="250" height="30"></canvas>
        <button id="saveButton">Save PNG</button>
    </div>
</div>

<script>
// --- PARAMETERS & SETUP ---
const canvas = document.getElementById('psycanvas');
const ctx = canvas.getContext('2d');
const paletteCanvas = document.getElementById('paletteCanvas');
const paletteCtx = paletteCanvas.getContext('2d');

let cellSize = 80;
let freqX = 5;
let freqY = 8;
let partCount = 8;
let trail = 45; // 0:none, 99: almost full decay
let baseColor = "#ff4eb6";

let W = canvas.width, H = canvas.height;

// Responsive design: adjust size.
function fitCanvas() {
    if(window.innerWidth < 700) {
        canvas.width = Math.floor(window.innerWidth*0.97);
        canvas.height = Math.floor(window.innerWidth*0.80);
    } else {
        canvas.width = W; canvas.height = H;
    }
}
fitCanvas();
window.addEventListener('resize', fitCanvas);

// Utilities
function lerp(a,b,t){ return a+(b-a)*t; }

function hexToRgb(h) {
    let n = parseInt(h.slice(1), 16);
    return {r:(n>>16)&255,g:(n>>8)&255,b:n&255};
}

function rgbToHex(c) {
    return '#' + [c.r,c.g,c.b].map(x=>x.toString(16).padStart(2,'0')).join('');
}

function rotateHue(rgb, deg) {
    // Rotate hue of rgb by deg degrees (simple rgb->hsv->rgb).
    // from https://stackoverflow.com/a/54070620
    let r=rgb.r, g=rgb.g, b=rgb.b;
    let h,s,v;
    let min = Math.min(r,g,b), max=Math.max(r,g,b);
    v = max;
    let d = max-min;
    s = max===0?0:d/max;
    if(max === min) h = 0;
    else {
        switch(max){
            case r: h = (g-b)/d + (g<b?6:0); break;
            case g: h = (b-r)/d +2; break;
            case b: h = (r-g)/d +4; break;
        }
        h/=6;
    }
    h = (h + deg/360) % 1;
    if(h<0) h+=1;
    let i = Math.floor(h*6);
    let f = h*6-i;
    let p = v*(1-s);
    let q = v*(1-f*s);
    let t = v*(1-(1-f)*s);
    switch(i%6){
      case 0: r=v,g=t,b=p; break;
      case 1: r=q,g=v,b=p; break;
      case 2: r=p,g=v,b=t; break;
      case 3: r=p,g=q,b=v; break;
      case 4: r=t,g=p,b=v; break;
      case 5: r=v,g=p,b=q; break;
    }
    return { r:Math.round(r), g:Math.round(g), b:Math.round(b) };
}

// Generate a palette (array of N rgb colors) around a base
function generatePsyPalette(base, count) {
    // base is hex or rgb; returns rgb[]
    let c = typeof(base)==='string'?hexToRgb(base):base;
    let palette = [];
    for(let i=0; i<count; ++i) {
        let hrot = (360*i/count);
        let brightShift = (Math.sin(i*Math.PI*2/count + Math.PI/3)+1)*0.2 + 0.7;
        let color = rotateHue(c, hrot);
        color.r = Math.min(255, Math.round(color.r*brightShift));
        color.g = Math.min(255, Math.round(color.g*brightShift));
        color.b = Math.min(255, Math.round(color.b*brightShift));
        palette.push(color);
    }
    return palette;
}

// Update palette gradient display
function drawPaletteBar() {
    let n = 30;
    let pal = generatePsyPalette(baseColor, n);
    paletteCtx.clearRect(0,0,250,30);
    for(let i=0; i<n; ++i){
        paletteCtx.fillStyle = `rgb(${pal[i].r},${pal[i].g},${pal[i].b})`;
        paletteCtx.fillRect(i*250/n, 0, 250/n+1, 30);
    }
}

// --- CONTROL BINDINGS ---
function bindRange(id, min, max, val, cb) {
    let ele = document.getElementById(id);
    ele.min=min; ele.max=max; ele.value=val;
    document.getElementById(id+'-value').innerText = val;
    ele.addEventListener('input', function() {
        cb(Number(ele.value));
        document.getElementById(id+'-value').innerText = ele.value;
    });
}
bindRange("cellsize", 48, 160, cellSize, v=>cellSize=v);
bindRange("freqx", 1, 18, freqX, v=>freqX=v);
bindRange("freqy", 1, 18, freqY, v=>freqY=v);
bindRange("particles", 2, 12, partCount, v=>partCount=v);
bindRange("trail", 0, 99, trail, v=>trail=v);

let colorInput = document.getElementById('palette');
colorInput.value = baseColor;
colorInput.addEventListener('input', function(){
    baseColor = this.value;
    drawPaletteBar();
});
drawPaletteBar();

// --- Lissajous Patterned Particles ---
let wavePhase = 0;

function drawFrame(ts) {
    let w=canvas.width, h=canvas.height;
    // Fade (psychedelic paper trail effect)
    if(trail<98) {
        ctx.globalAlpha = 1- trail/102;
        ctx.fillStyle="#150f1c";
        ctx.fillRect(0,0,w,h);
        ctx.globalAlpha=1;
    } else {
        ctx.clearRect(0,0,w,h);
    }
  
    let gridCols = Math.ceil(w/cellSize);
    let gridRows = Math.ceil(h/cellSize);
    let pp = generatePsyPalette(baseColor, partCount);

    // Animate phase for constant flow
    wavePhase += 0.008 + 0.003 * Math.sin(ts/7649);
    let t = ts*0.00026;

    for(let r=0; r<gridRows; ++r) {
        for(let c=0; c<gridCols; ++c) {
            let cx = c*cellSize + cellSize/2;
            let cy = r*cellSize + cellSize/2;

            for(let i=0; i<partCount; ++i){
                // Lissajous parameters, modulated per particle
                let fx = freqX + i*Math.sin(t+0.42*i);
                let fy = freqY + i*Math.cos(t-0.72*i);
                let px = Math.cos((t+wavePhase)*fx + (i*1.6)) * cellSize*0.36;
                let py = Math.sin((t+wavePhase)*fy + (i*1.17)) * cellSize*0.36;

                // Encircling waveform effect, growing/shrinking
                let dilate = 0.48 + 0.18*Math.sin((t+0.81*i)+(c+r)*0.4);

                let x = cx+px*dilate;
                let y = cy+py*dilate;

                // Outline "ripple"
                ctx.lineWidth = lerp(1.6,3.25,Math.abs(Math.cos(i+t/2))/2.5);
                ctx.strokeStyle=`rgba(12,20,44,0.07)`;
                ctx.beginPath();
                ctx.arc(x, y, 13 + 7*Math.abs(Math.sin(t+i)), 0, Math.PI*2);
                ctx.stroke();

                // Particle glow
                let color = pp[i];
                let alpha = 0.68 + 0.24*Math.sin(i+t + c*0.7);
                ctx.save();
                ctx.beginPath();
                ctx.arc(x,y, 7+3*(Math.sin(i+t)*0.5+0.5),0,Math.PI*2);
                ctx.shadowColor=`rgba(${color.r},${color.g},${color.b},1)`;
                ctx.shadowBlur=28;
                ctx.fillStyle=`rgba(${color.r},${color.g},${color.b},${alpha})`;
                ctx.globalAlpha=0.78;
                ctx.fill();
                ctx.restore();

                // Inner nucleus
                ctx.beginPath();
                ctx.arc(x, y, 2.9+1.0*Math.sin(i+t+Math.PI), 0, Math.PI*2);
                ctx.fillStyle=`rgb(${lerp(35,color.r,0.5)},${lerp(44,color.g,0.5)},${lerp(50,color.b,0.5)})`;
                ctx.globalAlpha=1.0;
                ctx.fill();
            }

            // Animate waveforms within the grid cells
            ctx.save();
            ctx.translate(cx,cy);
            ctx.rotate(0.4*Math.sin(t+(c-r)*0.45));
            ctx.strokeStyle = "#aabbff11";
            ctx.lineWidth=1;
            ctx.beginPath();
            for(let s=0; s<cellSize; s+=2.5){
                let phase = ((c*r)%2===0)?t:-t;
                let vy = Math.sin((s/cellSize)*Math.PI*2*freqY + t + c*0.7)*5;
                ctx.lineTo(-cellSize/2+s, Math.sin((s/cellSize)*Math.PI*freqX + phase)*6 + vy);
            }
            ctx.globalAlpha=0.38;
            ctx.stroke();
            ctx.restore();
        }
    }
    window.requestAnimationFrame(drawFrame);
}
window.requestAnimationFrame(drawFrame);

// --- SAVE PNG FUNCTION ---
function saveCanvasAsImage(canvas) {
    const dataURL = canvas.toDataURL('image/png');
    const filename = 
        `psywave_grid_cell${cellSize}_fx${freqX}_fy${freqY}_parts${partCount}.png`;
    const link = document.createElement('a');
    link.href = dataURL;
    link.download = filename;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}
document.getElementById('saveButton').addEventListener('click', function() {
    saveCanvasAsImage(canvas);
});
</script>
