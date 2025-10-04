---
layout: fullscreen
title: Psychedelic Wave Orbits – Lissajous Spirals with Orbital Particle Trails
tags:
  - graphics
---

This variation fuses colorful Lissajous figures and reactive spiral particle trails into a mesmerizing, evolving "psychedelic wave orbits" animation. Each customizable control influences either the Lissajous path (curve shape), particle emission, or color palette. Select vivid base colors—watch the spiral waves and orbiting traces morph and pulse in entrancing harmony.

<style>
body {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: radial-gradient(circle at 50% 45%, #18172a 60%, #110d24 100%);
    min-height: 100vh;
    margin: 0;
    padding-top: 300px;
}
canvas {
    border: 2px solid #111;
    margin-top: 20px;
    box-shadow: 0 0 24px #0008;
    background: transparent;
}
.controls {
    margin-top: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
}
.control-group {
    margin: 8px 0;
    display: flex;
    align-items: center;
}
.control-group label {
    min-width: 135px;
    color: #fff;
    letter-spacing: 1px;
}
.value-label {
    margin-left: 10px;
    font-weight: bold;
    color: #9ef;
}
input[type="range"] {
    width: 150px;
}
.color-spectrum {
    margin: 10px 0;
    width: 320px;
}
</style>
<canvas id="canvas" width="650" height="650"></canvas>

<div class="controls">
  <div class="control-group">
      <label for="ax">Freq X (ax):</label>
      <input type="range" id="ax" min="1" max="14" value="6">
      <span id="ax-value" class="value-label">6</span>
  </div>
  <div class="control-group">
      <label for="ay">Freq Y (ay):</label>
      <input type="range" id="ay" min="1" max="14" value="7">
      <span id="ay-value" class="value-label">7</span>
  </div>
  <div class="control-group">
      <label for="delta">Phase (Δ):</label>
      <input type="range" id="delta" min="0" max="628" value="157">
      <span id="delta-value" class="value-label">0.25π</span>
  </div>
  <div class="control-group">
      <label for="amp">Amplitude:</label>
      <input type="range" id="amp" min="100" max="290" value="220">
      <span id="amp-value" class="value-label">220</span>
  </div>
  <div class="control-group">
      <label for="trail">Trail Length:</label>
      <input type="range" id="trail" min="15" max="120" value="42">
      <span id="trail-value" class="value-label">42</span>
  </div>
  <div class="control-group">
      <label for="pcount">Particles:</label>
      <input type="range" id="pcount" min="24" max="80" value="48">
      <span id="pcount-value" class="value-label">48</span>
  </div>
  <div class="control-group">
      <label for="colorbase">Base Color:</label>
      <input type="color" id="colorbase" value="#32f0ec">
  </div>
  <canvas id="gradientCanvas" width="320" height="40" class="color-spectrum"></canvas>
</div>

<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const gc = document.getElementById('gradientCanvas');
const gctx = gc.getContext('2d');

let ax = 6, ay = 7, amp = 220, delta = Math.PI/4, 
    numParticles = 48, trailLen = 42, 
    baseColor = "#32f0ec";
function displayPhaseLabel(v) {
    let n = (parseInt(v)/Math.PI).toFixed(2);
    if(Math.abs(n-0.5)<0.01)return "0.5π";
    if(Math.abs(n-1)<0.01)return "π";
    return (parseFloat(v)/Math.PI).toFixed(2) + "π";
}

function hexToRgb(hex) {
    let bigint = parseInt(hex.slice(1),16);
    return {
        r:(bigint>>16)&255,
        g:(bigint>>8)&255,
        b:bigint&255
    };
}
function lerp(a,b,t){return a+(b-a)*t;}
function rgb(r,g,b){
    return `rgb(${Math.round(r)},${Math.round(g)},${Math.round(b)})`;
}
function genColorArray(base, N, vib=0.65, shift=0){
    // Generates N rainbow hues around base color (vib: vibrance factor, shift: hue offset)
    let c = hexToRgb(base);
    let [h0,s0,v0]=rgb2hsv(c.r,c.g,c.b);
    let arr = [];
    for(let i=0;i<N;i++){
        let h = (h0 + shift + i/N)%1;
        arr.push(hsv2rgb(
            h,
            lerp(s0, vib, 0.6),
            lerp(v0, 1.0, 0.6)
        ));
    }
    return arr;
}
function rgb2hsv(r,g,b){
    r/=255;g/=255;b/=255;
    let max = Math.max(r,g,b), min=Math.min(r,g,b), d=max-min,s,v=max;
    if(!d)return [0,0,v];
    s=d/max;
    let h;
    switch(max){
        case r: h=(g-b)/d + (g<b?6:0);break;
        case g: h=2+(b-r)/d;break;
        case b: h=4+(r-g)/d;break;
    }
    h/=6;
    return [h,s,v];
}
function hsv2rgb(h,s,v){
    let i=Math.floor(h*6),f=h*6-i,p=v*(1-s),q=v*(1-f*s),t=v*(1-(1-f)*s);
    let mod = i%6;
    let map = [[v,t,p],[q,v,p],[p,v,t],[p,q,v],[t,p,v],[v,p,q]];
    let [r,g,b] = map[mod];
    return rgb(r*255,g*255,b*255);
}

function drawColorGradient() {
    let cols = genColorArray(baseColor,32,0.95,0.07);
    let w=gc.width, h=gc.height;
    let grad = gctx.createLinearGradient(0,0,w,0);
    cols.forEach((color,i)=>{
        grad.addColorStop(i/(cols.length-1), color);
    });
    gctx.clearRect(0,0,w,h);
    gctx.fillStyle=grad;
    gctx.fillRect(0,0,w,h);
}

// --- Lissajous Wave Path & Particles ---
let t = 0, dt = 0.012;
let spiralParticles = [];
function resetParticles(){
    spiralParticles = [];
    for(let i=0;i<numParticles;i++){
        let theta = i * 2*Math.PI/numParticles;
        spiralParticles.push({
            theta,
            trail: Array.from({length:trailLen},(_)=>{
                return {x:0,y:0}
            }),
            colorIndex: i
        });
    }
}
resetParticles();

function draw() {
    ctx.globalAlpha = 1;
    ctx.clearRect(0,0,canvas.width,canvas.height);

    const colorWave = genColorArray(baseColor,numParticles, 0.80, 0.1);
    const colorTrail = genColorArray(baseColor, trailLen, 0.76, 0.32);

    // Draw spiral Lissajous path
    let cx = canvas.width/2, cy = canvas.height/2;
    ctx.save();
    ctx.beginPath();
    for(let alpha=0;alpha<2*Math.PI;alpha+=0.0075){
        let rad = amp * (0.60 + 0.37*Math.sin(4*alpha+t*1.2)); // modulated radius
        let x = cx + rad * Math.sin(ax*alpha+Math.cos(t*0.7));
        let y = cy + rad * Math.sin(ay*alpha+delta);
        if(alpha===0)ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
    }
    ctx.strokeStyle = ctx.createLinearGradient(cx-amp,cy-amp,cx+amp,cy+amp);
    let gcol = genColorArray(baseColor,4,0.85,0.02);
    for(let i=0;i<gcol.length;i++){
        ctx.strokeStyle.addColorStop(i/(gcol.length-1),gcol[i]);
    }
    ctx.lineWidth = 2.25;
    ctx.shadowColor = '#44fff4';
    ctx.shadowBlur = 18;
    ctx.globalAlpha = 0.80;
    ctx.stroke();
    ctx.restore();

    // Evolving particles with orbital radii + Lissajous-affected trails
    spiralParticles.forEach((p, idx)=>{
        let baseTheta = p.theta + t*0.43;
        // Particle core motion follows the spiral + Lissajous
        let rmod = amp*0.34 + 80*Math.sin(t*0.18+idx);
        let px = cx + (rmod + 48*Math.sin(baseTheta*ay+0.8*t)) * Math.sin(ax*baseTheta+t*0.3);
        let py = cy + (rmod + 59*Math.cos(baseTheta*ax-delta*1.5)) * Math.sin(ay*baseTheta+delta);
        // Maintain trail
        p.trail.unshift({x:px,y:py});
        if(p.trail.length>trailLen) p.trail.pop();

        // --- Draw particle trails
        ctx.save();
        ctx.beginPath();
        let fade = 1;
        for(let j=0;j<p.trail.length-1;j++){
            let pt0=p.trail[j], pt1=p.trail[j+1];
            let tfrac = j/(p.trail.length-1);
            ctx.strokeStyle = colorTrail[j];
            ctx.globalAlpha = lerp(0.84,0.015, tfrac**1.3);
            ctx.lineWidth = lerp(3.8, 0.3, tfrac**1.1);
            ctx.beginPath();
            ctx.moveTo(pt0.x,pt0.y);
            ctx.lineTo(pt1.x,pt1.y);
            ctx.stroke();
        }
        ctx.restore();

        // --- Draw glowing orbiting particle
        ctx.save();
        ctx.globalAlpha = 0.95;
        ctx.beginPath();
        ctx.arc(px,py, lerp(6.4,2.2,Math.cos(baseTheta+t*0.71)*0.53+0.3), 0, Math.PI*2);
        ctx.fillStyle = colorWave[p.colorIndex];
        ctx.shadowBlur = lerp(12, 26, Math.cos(idx*0.22+t*1.7)*0.60+0.4);
        ctx.shadowColor = colorTrail[Math.floor(lerp(0, trailLen-1, 0.18+0.8*Math.sin(idx*0.11+t*1.1)))];
        ctx.fill();
        ctx.restore();
    });

    t += dt;
    requestAnimationFrame(draw);
}

// ---- Controls binding ----
document.getElementById('ax').addEventListener('input', function(){
    ax = parseInt(this.value);
    document.getElementById('ax-value').innerText = this.value;
});
document.getElementById('ay').addEventListener('input', function(){
    ay = parseInt(this.value);
    document.getElementById('ay-value').innerText = this.value;
});
document.getElementById('delta').addEventListener('input', function(){
    delta = parseInt(this.value)/200;
    let label = (parseInt(this.value)/Math.PI).toFixed(2)+"π";
    document.getElementById('delta-value').innerText = displayPhaseLabel(delta);
});
document.getElementById('amp').addEventListener('input', function(){
    amp = parseInt(this.value);
    document.getElementById('amp-value').innerText = this.value;
});
document.getElementById('trail').addEventListener('input', function(){
    trailLen = parseInt(this.value);
    document.getElementById('trail-value').innerText = this.value;
    resetParticles();
});
document.getElementById('pcount').addEventListener('input', function(){
    numParticles = parseInt(this.value);
    document.getElementById('pcount-value').innerText = this.value;
    resetParticles();
});
document.getElementById('colorbase').addEventListener('input', function(){
    baseColor = this.value;
    drawColorGradient();
});

drawColorGradient();
draw();
</script>
