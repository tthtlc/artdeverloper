```markdown
---
layout: fullscreen
title: "Quantum Waves: Interactive Lissajous Particle Tunnels"
tags:
  - graphics
---

<style>
    canvas {
        border: 1px solid black;
        background: #161624;
        display: block;
        margin: 0 auto;
    }
    .controls {
        margin-top: 24px;
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 8px;
    }
    .control-group {
        display: flex;
        align-items: center;
        gap: 12px;
    }
    .control-group label { color: #eee; }
    .sublabel {
        font-size: 11px;
        color: #aaa;
        margin-left: 3px;
    }
    input[type="range"] { width: 160px; }
    input[type="color"] { border: none; padding: 0; width: 32px; height: 24px;}
    .color-preview {
        width: 24px; height: 24px; border-radius: 4px; display: inline-block; margin-left: 6px; border: 1px solid #ccc;
        box-shadow: 0 0 2px rgba(0,0,0,0.4);
    }
    button { margin-top: 8px; padding: 6px 18px; font-size: 14px; border-radius: 4px; }
</style>
<canvas id="canvas" width="900" height="600"></canvas>

<div class="controls">
    <div class="control-group">
        <label for="aFreq">x freq:</label>
        <input type="range" id="aFreq" min="1" max="12" value="3">
        <span id="aFreq-value" class="value-label">3</span>
        <span class="sublabel">(waves X)</span>
    </div>
    <div class="control-group">
        <label for="bFreq">y freq:</label>
        <input type="range" id="bFreq" min="1" max="12" value="2">
        <span id="bFreq-value" class="value-label">2</span>
        <span class="sublabel">(waves Y)</span>
    </div>
    <div class="control-group">
        <label for="depth">Tunnel depth:</label>
        <input type="range" id="depth" min="6" max="80" value="40">
        <span id="depth-value" class="value-label">40</span>
    </div>
    <div class="control-group">
        <label for="count">Particle count:</label>
        <input type="range" id="count" min="10" max="120" value="48">
        <span id="count-value" class="value-label">48</span>
    </div>
    <div class="control-group">
        <label for="saturation">Color Sat:</label>
        <input type="range" id="saturation" min="10" max="100" value="88">
        <span id="saturation-value" class="value-label">88%</span>
        <span class="sublabel">(HSL)</span>
    </div>
    <div class="control-group">
        <label for="baseHue">Base Hue:</label>
        <input type="range" id="baseHue" min="0" max="360" value="225">
        <span id="baseHue-value" class="value-label">225°</span>
        <input type="color" id="colorPick" value="#4f4fff">
        <span class="color-preview" id="colorSample" style="background: hsl(225,88%,50%)"></span>
    </div>
    <div class="control-group">
        <label for="trailLength">Trail:</label>
        <input type="range" id="trailLength" min="12" max="80" value="36">
        <span id="trailLength-value" class="value-label">36</span>
        <span class="sublabel">(motion echo)</span>
    </div>
    <button id="saveButton">Save as PNG</button>
</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let W = canvas.width, H = canvas.height;
let cx = W/2, cy = H/2;

// Controls & State
let params = {
    aFreq: 3,
    bFreq: 2,
    depth: 40,
    count: 48,
    baseHue: 225,
    saturation: 88,
    trailLength: 36,
    colorPick: "#4f4fff"
};
let lastHue = params.baseHue;

// === Controls Handling ===
function updateParams(id, value) {
    value = parseInt(value);
    params[id] = value;
    document.getElementById(id + '-value').innerText = 
        id === 'baseHue' ? value + "°" :
        id === 'saturation' ? value + "%" :
        value;
    if (id == 'baseHue' || id == 'saturation') {
        // update color preview
        document.getElementById("colorSample").style.background = `hsl(${params.baseHue},${params.saturation}%,50%)`;
        // update color pick value
        let c = hsl2hex(params.baseHue, params.saturation, 50);
        document.getElementById("colorPick").value = c;
        params.colorPick = c;
    }
}
[["aFreq",1,12],["bFreq",1,12],["depth",6,80],["count",10,120],["saturation",10,100],["baseHue",0,360],["trailLength",12,80]].forEach(([id,min,max])=>{
    document.getElementById(id).addEventListener('input',e=>{
        updateParams(id,e.target.value);
    });
});
document.getElementById("colorPick").addEventListener("input",e=>{
    let rgb = hexToRgb(e.target.value), hsl = rgbToHsl(rgb.r,rgb.g,rgb.b);
    params.baseHue = Math.round(hsl[0]*360);
    params.saturation = Math.round(hsl[1]*100);
    updateParams("baseHue", params.baseHue);
    updateParams("saturation", params.saturation);
});
updateParams("baseHue", params.baseHue);
updateParams("saturation", params.saturation);

// === Utility color functions ===
function hsl2hex(h,s,l) {
    h/=360; s/=100; l/=100;
    let r, g, b;
    if(s==0){
        r=g=b=l;
    }else{
        let hue2rgb = (p, q, t)=>{
            if(t<0) t+=1;
            if(t>1) t-=1;
            if(t<1/6) return p+(q-p)*6*t;
            if(t<1/2) return q;
            if(t<2/3) return p+(q-p)*(2/3-t)*6;
            return p;
        };
        let q = l<0.5 ? l * (1+s) : l + s - l*s;
        let p = 2*l-q;
        r = hue2rgb(p,q,h+1/3);
        g = hue2rgb(p,q,h);
        b = hue2rgb(p,q,h-1/3);
    }
    return `#${[r,g,b].map(x=>(Math.round(x*255).toString(16).padStart(2,'0'))).join('')}`;
}
function hexToRgb(hex) {
    hex = hex.replace(/^#/, '');
    if (hex.length === 3) hex = hex.split('').map(x => x+x).join('');
    let int = parseInt(hex, 16);
    return {r:(int>>16)&255, g:(int>>8)&255, b:int&255};
}
function rgbToHsl(r, g, b) {
    r/=255, g/=255, b/=255;
    let max=Math.max(r,g,b), min=Math.min(r,g,b), h,s,l=(max+min)/2;
    if(max==min){h=s=0;}else{
        let d=max-min;
        s=l>0.5?d/(2-max-min):d/(max+min);
        switch(max){
            case r: h=(g-b)/d+(g<b?6:0); break;
            case g: h=(b-r)/d+2; break;
            case b: h=(r-g)/d+4; break;
        }
        h/=6;
    }
    return [h,s,l];
}

// === Tunnel + Particle System ===

let time = 0;
const TAU = Math.PI*2;

function quantumColor(theta, t, i) {
    // Generate evolving neon hues with a stationary base
    let hue = (params.baseHue + 
            120*Math.sin(theta + t*0.71 + i) + 
            120*Math.sin(3*theta + t*0.43)
        )%360;
    return `hsl(${hue<0?hue+360:hue},${params.saturation}%,54%)`;
}

function draw() {
    ctx.globalCompositeOperation = "source-over";
    // Partial fade for trail
    ctx.fillStyle = `rgba(22,22,36,${1 - params.trailLength/100})`;
    ctx.fillRect(0,0,W,H);

    // Tunnel parameters
    const particles = params.count;
    const layers = params.depth;
    const freqX = params.aFreq;
    const freqY = params.bFreq;
    let particleTrails = 4 + Math.max(2, Math.round(params.trailLength*0.3)); // echo dots per particle
    let orbR = Math.max(7,40 - layers*0.4); // size of layers

    // Animate depth undulation
    let depthMod = Math.sin(time*0.19)*0.45 + 1;

    // Draw tunnel, front to back
    for (let l=layers-1; l>=0; --l) {
        let scale = (1-0.92*l/layers)*1.5 + 0.2*depthMod;
        let z = l/layers;
        let opacity = Math.max(0.12, 1.126 - z*1.21);

        let wavePhase = l*0.13 + time*0.048;
        let layerFreqPhase = Math.PI/4*Math.sin(time*0.014 + l*0.02);

        for (let p=0; p<particles; ++p) {
            // Lissajous-like oscillation with slow rotation twist
            let waveTheta = (TAU*p/particles) + wavePhase;
            let phase = time*0.03 + l*0.17;
            let x = cx + scale * (
                210 * Math.sin(freqX*waveTheta + phase + Math.sin(time*0.06+l*0.3)) +
                70*Math.cos(waveTheta + layerFreqPhase)
            );
            let y = cy + scale * (
                210 * Math.sin(freqY*waveTheta + phase + Math.cos(time*0.05+l*0.3)) +
                70*Math.sin(waveTheta + layerFreqPhase)
            );
            
            // Color spectrum tunnel effect
            let col = quantumColor(waveTheta, time, l*0.07);

            // "Particle glow" - trail behind
            for (let tEcho = particleTrails; tEcho>0; --tEcho) {
                let fade = tEcho/particleTrails; // 1 down to 0
                let trailPhase = phase - fade*0.28 - z*0.1;
                let tx = cx + scale * (
                    210 * Math.sin(freqX*waveTheta + trailPhase + Math.sin(time*0.06+l*0.3)) +
                    70*Math.cos(waveTheta + layerFreqPhase)
                );
                let ty = cy + scale * (
                    210 * Math.sin(freqY*waveTheta + trailPhase + Math.cos(time*0.05+l*0.3)) +
                    70*Math.sin(waveTheta + layerFreqPhase)
                );
                let s = orbR*fade*(0.48+0.3*Math.abs(Math.sin(tEcho+time*0.09)))*(0.7+z*0.2);
                ctx.globalAlpha = opacity*fade*0.19;
                ctx.beginPath();
                ctx.arc(tx,ty, s, 0, TAU);
                ctx.fillStyle = col;
                ctx.shadowColor = col;
                ctx.shadowBlur = 16*fade*fade + 2;
                ctx.fill();
            }

            // Main particle (sharp)
            ctx.globalAlpha = opacity;
            ctx.beginPath();
            ctx.arc(x,y, orbR*0.6*(1-z*0.6), 0, TAU);
            ctx.fillStyle = col;
            ctx.shadowColor = col;
            ctx.shadowBlur = 16;
            ctx.fill();
        }
    }
    ctx.globalAlpha = 1.0;
    ctx.shadowBlur = 0;
}

function animate() {
    time += 1;
    draw();
    requestAnimationFrame(animate);
}

// Start!
draw();
animate();

// --- Resize support ---
window.addEventListener('resize', ()=>{
    let box = canvas.getBoundingClientRect();
    // Try to keep aspect
    W = window.innerWidth * 0.97;
    H = window.innerHeight * 0.74;
    // cap at (1200,800)
    if(W>1200) W=1200;
    if(H>800) H=800;
    canvas.width = W; canvas.height = H;
    cx = W/2; cy = H/2;
});

// --- Save as PNG ---
document.getElementById("saveButton").addEventListener('click',function(){
    let filename = `QuantumWaves_LissajousTunnel_x${params.aFreq}_y${params.bFreq}_d${params.depth}_h${params.baseHue}.png`;
    let link = document.createElement('a');
    link.download = filename;
    link.href = canvas.toDataURL('image/png');
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
});

// Initial resize for full window
window.dispatchEvent(new Event("resize"));
</script>
```
