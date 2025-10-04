---
layout: fullscreen
title: "Psychedelic Flowerwave"
tags:
  - graphics
---

<style>
canvas {
  background: radial-gradient(ellipse at center, #1a0033 0%, #000016 100%);
  display: block;
  margin: 0 auto 30px auto;
  border: 2px solid #222;
}
.controls {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 10px;
    align-items: center;
    width: 80%;
    margin: 0 auto 20px auto;
}
</style>

<canvas id="flowerwave" width="800" height="800"></canvas>
<div class="controls">
  <label for="petals">Petals:</label>
  <input type="range" id="petals" min="3" max="24" value="8">
  <span id="petalsValue">8</span>
  
  <label for="layers">Layers:</label>
  <input type="range" id="layers" min="1" max="7" value="4">
  <span id="layersValue">4</span>
  
  <label for="ampl">Wave Amplitude:</label>
  <input type="range" id="ampl" min="10" max="200" value="70">
  <span id="amplValue">70</span>
  
  <label for="freq">Wave Freq:</label>
  <input type="range" id="freq" min="2" max="30" value="8">
  <span id="freqValue">8</span>
  
  <label for="speed">Speed:</label>
  <input type="range" id="speed" min="1" max="100" value="40">
  <span id="speedValue">40</span>
</div>

<script>
const canvas = document.getElementById('flowerwave');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;
const Cx = W/2, Cy = H/2;

let settings = {
  petals: 8,
  layers: 4,
  ampl: 70,
  freq: 8,
  speed: 40
};

const sliders = [
  ['petals', 'petalsValue'],
  ['layers', 'layersValue'],
  ['ampl', 'amplValue'],
  ['freq', 'freqValue'],
  ['speed', 'speedValue']
];

sliders.forEach(([id, valId]) => {
  document.getElementById(id).addEventListener('input', e => {
    settings[id] = parseInt(e.target.value);
    document.getElementById(valId).textContent = settings[id];
  });
  document.getElementById(valId).textContent = document.getElementById(id).value;
});

function hslColor(h, s, l) {
  return `hsl(${h}, ${s}%, ${l}%)`;
}

// Generate a psychedelic color for each point
function getColor(i, t, layer, petalCount, totalPoints) {
  const h = (Math.sin(t/650 + i*0.12 + layer*0.18) * 60 + 240 + 360) % 360;
  const s = 70 + 25 * Math.sin(i*0.23 + layer*t*0.015);
  const l = 60 + 25 * Math.cos(t*0.008 + layer + i*0.13);
  return hslColor(h, s, l);
}

function drawFlowerwave(t) {
  ctx.clearRect(0,0,W,H);
  ctx.save();
  ctx.translate(Cx, Cy);

  const petals = settings.petals;
  const layers = settings.layers;
  const ampl = settings.ampl;
  const freq = settings.freq;
  const speed = settings.speed/70;

  for(let layer=0; layer<layers; layer++) {
    ctx.beginPath();
    let totalPoints = 360;
    const R = 170 + 45*Math.sin(t/1300 + layer*1.95); // Base radius
  
    for(let i=0; i<=totalPoints; i++) {
      const theta = 2*Math.PI * i / totalPoints;
      // Layer progression causes shearing/offset
      const wphase = t*speed*0.007 + layer*1.3;
      // Funky wave modulation
      const flowerMod = Math.sin(petals*theta + wphase + Math.cos(freq*theta + t*0.0012 + layer));
      const alt = Math.cos(layer*0.9 + theta*petals/2 + wphase/2);
      const radius = R 
        + ampl * flowerMod
        + ampl*(0.25 + 0.19*layer) * alt * Math.sin(freq*theta + t*0.0031 - layer);
    
      const x = radius * Math.cos(theta);
      const y = radius * Math.sin(theta);

      if(i===0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);
    }
    ctx.closePath();

    // Rainbow glow
    ctx.shadowColor = getColor(layer * 60 + 80, t, layer, petals, totalPoints);
    ctx.shadowBlur = 33 + 10*layer;

    ctx.lineWidth = 3.3 + 0.7*layer;
    ctx.strokeStyle = getColor(layer * 61, t, layer, petals, totalPoints);
    ctx.stroke();

    // Fill with gradient
    let grad = ctx.createRadialGradient(0,0,60, 0,0,R+ampl);
    grad.addColorStop(0, 'rgba(255,255,255,0.04)');
    grad.addColorStop(1, getColor(360-layer*44, t, layer, petals, totalPoints).replace('%)', '%, .12)'));
    ctx.fillStyle = grad;
    ctx.globalAlpha = 0.23 + 0.18*Math.abs(Math.sin(t/900 + layer));
    ctx.fill();
    ctx.globalAlpha = 1.0;
  }
  ctx.restore();
}

let startTime = null;
function animate(time){
  if(!startTime) startTime = time;
  const t = time - startTime;
  drawFlowerwave(t);
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

</script>
