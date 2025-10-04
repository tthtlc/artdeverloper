---
layout: fullscreen
title: "Kaleidoscopic Lissajous Particles with Wave Modulation and Controls"
tags:
  - graphics
---

<style>
canvas {
  border: 1px solid #333;
  background: #111;
  display: block;
  margin: 0 auto;
}
.controls {
  margin-top: 24px;
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
  margin-right: 10px;
  color: #fff;
  min-width: 120px;
}
input[type="range"] {
  width: 180px;
}
.value-label {
  margin-left: 10px;
  font-weight: bold;
  color: #fff;
  width: 40px;
  display: inline-block;
}
</style>
<canvas id="canvas" width="700" height="700"></canvas>
<div class="controls">
  <div class="control-group">
    <label for="freqX">X Frequency:</label>
    <input type="range" min="1" max="12" id="freqX" value="4"/>
    <span id="freqX-value" class="value-label">4</span>
  </div>
  <div class="control-group">
    <label for="freqY">Y Frequency:</label>
    <input type="range" min="1" max="12" id="freqY" value="7"/>
    <span id="freqY-value" class="value-label">7</span>
  </div>
  <div class="control-group">
    <label for="numParticles">Particles:</label>
    <input type="range" min="12" max="220" id="numParticles" value="120"/>
    <span id="numParticles-value" class="value-label">120</span>
  </div>
  <div class="control-group">
    <label for="kaleido">Kaleidoscope:</label>
    <input type="range" min="2" max="12" id="kaleido" value="6"/>
    <span id="kaleido-value" class="value-label">6</span>
  </div>
  <div class="control-group">
    <label for="amplitude">Wave Amplitude:</label>
    <input type="range" min="10" max="200" id="amplitude" value="80"/>
    <span id="amplitude-value" class="value-label">80</span>
  </div>
  <div class="control-group">
    <label for="palette">Base Color:</label>
    <input type="color" id="palette" value="#00fff3"/>
  </div>
</div>
<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
let width = canvas.width, height = canvas.height;

let freqX = 4;
let freqY = 7;
let numParticles = 120;
let kaleido = 6;
let amplitude = 80;
let baseColor = "#00fff3";

// Handle window resize
window.addEventListener('resize',()=> {
  const size = Math.min(window.innerWidth, window.innerHeight) - 48;
  canvas.width = canvas.height = Math.max(300, Math.min(size, 900));
  width = canvas.width;
  height = canvas.height;
});

let time = 0;
function draw() {
  // Fading trail effect for psychedelia
  ctx.globalAlpha = 0.12;
  ctx.fillStyle = "#111";
  ctx.fillRect(0,0,width,height);
  ctx.globalAlpha = 1;

  const cx = width/2, cy = height/2;
  const radius = Math.min(width, height) * 0.42;

  // Prepare color palette gradient based on baseColor
  const base = hexToHSL(baseColor);
  const palette = (i) => {
    return `hsl(${(base.h + 360*i/numParticles)%360}, ${Math.round(base.s*100)}%, ${55+25*Math.sin((i/numParticles)*Math.PI)}%)`;
  };

  for(let k=0; k<kaleido; ++k) {
    const angleK = (2*Math.PI * k)/kaleido;
    ctx.save();
    ctx.translate(cx, cy);
    ctx.rotate(angleK);
    for(let i=0;i<numParticles;++i) {
      // Lissajous curve parameters
      const t = time*0.012 + i*2*Math.PI/numParticles;
      let x = radius * Math.sin(freqX * t + 0.3*Math.sin(time*0.008+i)) / (1.1+0.5*Math.sin(i+time*0.0017));
      let y = radius * Math.sin(freqY * t - 0.2*Math.cos(time*0.011-i));

      // Modulate radius with a moving inward wave to create dynamic ripples
      let modAmp = amplitude * Math.sin(time*0.021 + i*2.7) * Math.cos(t-(i*time*0.0005));
      let ax = (x/(radius))*modAmp;
      let ay = (y/(radius))*modAmp;

      ctx.beginPath();
      ctx.arc(x+ax, y+ay, 2.1 + Math.sin(time*0.032+i)*1.2, 0, 2*Math.PI);
      ctx.fillStyle = palette(i);
      ctx.shadowColor = palette(numParticles-i);
      ctx.shadowBlur = 25 + 12*Math.sin(i+0.2*time);
      ctx.fill();
      ctx.shadowBlur = 0;
    }
    ctx.restore();
  }

  time++;
  requestAnimationFrame(draw);
}

// Color utils
function hexToHSL(hex) {
  let r = 0, g = 0, b = 0;
  if(hex.length === 7){
    r = parseInt(hex.slice(1,3),16)/255;
    g = parseInt(hex.slice(3,5),16)/255;
    b = parseInt(hex.slice(5,7),16)/255;
  }
  const mx=Math.max(r,g,b),mn=Math.min(r,g,b);
  let h,s,l=(mx+mn)/2;
  if(mx===mn){h=s=0;}
  else{
    const d=mx-mn;
    s=l>0.5?d/(2-mx-mn):d/(mx+mn);
    switch(mx){
      case r: h=(g-b)/d+(g<b?6:0); break;
      case g: h=(b-r)/d+2; break;
      case b: h=(r-g)/d+4; break;
    }
    h*=60;
  }
  return {h,s,l};
}

// Controls
function updateVal(id, v) {
  document.getElementById(id+"-value").textContent = v;
}
document.getElementById('freqX').addEventListener('input', e=>{
  freqX = parseInt(e.target.value);
  updateVal('freqX', freqX);
});
document.getElementById('freqY').addEventListener('input', e=>{
  freqY = parseInt(e.target.value);
  updateVal('freqY', freqY);
});
document.getElementById('numParticles').addEventListener('input', e=>{
  numParticles = parseInt(e.target.value);
  updateVal('numParticles', numParticles);
});
document.getElementById('kaleido').addEventListener('input', e=>{
  kaleido = parseInt(e.target.value);
  updateVal('kaleido', kaleido);
});
document.getElementById('amplitude').addEventListener('input', e=>{
  amplitude = parseInt(e.target.value);
  updateVal('amplitude', amplitude);
});
document.getElementById('palette').addEventListener('input', e=>{
  baseColor = e.target.value;
});

// Set initial values
['freqX','freqY','numParticles','kaleido','amplitude'].forEach(id=>{
  updateVal(id, document.getElementById(id).value);
});

window.dispatchEvent(new Event('resize'));
draw();
</script>
