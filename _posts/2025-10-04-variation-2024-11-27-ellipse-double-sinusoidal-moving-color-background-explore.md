---
layout: fullscreen
title: "Psychedelic Oscillating Wave Mandala"
tags:
  - graphics
---

Hypnotic mandala formed by oscillating waveforms, evolving color, multi-armed rotating symmetry, and interactive control.

<style>
  canvas {
    border: 1px solid #111;
    background: #101018;
    display: block;
    margin:0 auto;
    box-shadow: 0 0 16px #222;
  }
  .controls {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap:16px;
    margin: 20px auto 0;
    justify-content:center;
    background: #191924aa;
    padding: 12px 24px;
    border-radius: 12px;
  }
  .label {
    color: #ffe;
    font-size:1em;
    margin-right: 6px;
    letter-spacing:.05em;
  }
  input[type=range] {
    accent-color: #8ef;
    margin:0 4px;
  }
  .val {
    color:#c2fdde;
    min-width:2.3em;
    display:inline-block;
    text-align:right;
    margin-left:4px;
    font-family: monospace;
  }
  select, input[type=color] {
    margin-left:8px;
    background:#191924;
    color:#fed;
    border-radius:3px;
    border:1px solid #444;
    padding:2px 4px;
  }
</style>

<canvas id="mandalaCanvas" width="700" height="700"></canvas>
<div class="controls">
  <span class="label">Arms:</span>
  <input type="range" id="armsControl" min="4" max="24" value="12">
  <span id="armsVal" class="val">12</span>
  
  <span class="label">Detail:</span>
  <input type="range" id="detailControl" min="80" max="480" value="240">
  <span id="detailVal" class="val">240</span>
  
  <span class="label">Wave Depth:</span>
  <input type="range" id="depthControl" min="10" max="120" value="48">
  <span id="depthVal" class="val">48</span>
  
  <span class="label">Wave Complexity:</span>
  <input type="range" id="complexityControl" min="1" max="16" value="7">
  <span id="complexityVal" class="val">7</span>

  <span class="label">Speed:</span>
  <input type="range" id="speedControl" min="1" max="100" value="25">
  <span id="speedVal" class="val">1.0x</span>

  <span class="label">Palette:</span>
  <select id="paletteSelector">
    <option value="ultraviolet">Ultraviolet</option>
    <option value="sunset">Sunset</option>
    <option value="neonrainbow">Neon Rainbow</option>
    <option value="wave">Psy Waves</option>
    <option value="candy">Candy Pop</option>
  </select>

  <span class="label">BG:</span>
  <input type="color" id="bgColorPicker" value="#101018">
</div>

<script>
const canvas = document.getElementById('mandalaCanvas');
const ctx = canvas.getContext('2d');

const width = canvas.width;
const height = canvas.height;
const centerX = width/2;
const centerY = height/2;

// Control knobs
let arms = 12;
let detail = 240;
let depth = 48;
let complexity = 7;
let animSpeed = 1.0;
let bgColor = "#101018";

// Palettes
const palettes = {
  ultraviolet: [
    "#fcf6f5", "#b38fff", "#7848ff", "#360568", "#521262",
    "#ff7edb", "#fff95b", "#ffb356"
  ],
  sunset: [
    "#ff5e62", "#ff9966", "#ffb88c", "#ffef78", "#8fd3f4", "#d9afd9"
  ],
  neonrainbow: [
    "#ff00c8", "#00ffe5", "#fff200", "#ff5e13", "#0500ff", "#b6ff00", "#ff073a"
  ],
  wave: [
    "#ad1cfd", "#20e3b2", "#f9c846", "#fb37a3", "#1ca7ec", "#ff1361"
  ],
  candy: [
    "#fca3cc","#ffdee9","#b5fffc","#bce5fa","#ffadad","#e6eeff","#baffc9","#d9e7fd"
  ]
};
let curPalette = palettes.ultraviolet;

// Utility: Interpolate between two hex colors, t=0..1
function lerpColor(a, b, t) {
  const ah = parseInt(a.slice(1),16), bh=parseInt(b.slice(1),16);
  const ar=(ah>>16)&255, ag=(ah>>8)&255, ab=ah&255;
  const br=(bh>>16)&255, bg=(bh>>8)&255, bb=bh&255;
  const r = Math.round(ar + (br-ar)*t);
  const g = Math.round(ag + (bg-ag)*t);
  const b_ = Math.round(ab + (bb-ab)*t);
  return `rgb(${r},${g},${b_})`;
}

// Mandala waveform formula
function mandalaWave(theta, time, armIndex) {
  // Several layered sine/cos waves of different frequencies
  // The magic is the animated phase offset for armIndex and time, for evolving undulations
  let val = 0;
  for (let k=1; k<=complexity; ++k) {
    let phase = time*0.33 + armIndex*0.14 + k*0.43;
    val += Math.sin(k*theta + Math.sin(time*0.7 + armIndex*0.42 + k)*2.0 + phase) / k;
  }
  // Superimpose a slow undulating global wave
  val += 0.7 * Math.sin(theta*arms/3 + time*0.18 + armIndex);
  return val;
}

function drawMandala(time) {
  // Animate arms rotation and helpful for secondary motion
  let evolve = time*0.23;
  ctx.save();
  ctx.clearRect(0,0,width,height);
  ctx.globalAlpha = 1.0;
  ctx.fillStyle = bgColor;
  ctx.fillRect(0,0,width,height);

  // Draw faint afterimages using composite for glow
  ctx.globalAlpha = 0.12;
  for (let repeat=0; repeat<3; ++repeat) {
    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.rotate(evolve*.29*(repeat+1));
    for(let a=0; a<arms; ++a){
      ctx.save();
      ctx.rotate((2*Math.PI/arms)*a);

      // Build the radial waveform path
      ctx.beginPath();
      for(let i=0; i<=detail; i++){
        let t = i/detail * 2*Math.PI;
        let baseR = 185 + depth*0.15*Math.sin(t*3 + time*0.7+repeat) + 35*Math.sin(a+evolve);
        let wave = mandalaWave(t, time+repeat*0.93, a);
        let r = baseR + wave*depth * (0.88-0.26*repeat);
        if (r<0) r = 0;
        let x = Math.cos(t)*r;
        let y = Math.sin(t)*r;
        if(i===0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
      }
      ctx.closePath();

      // Color based on t and arm index: blend palette
      for(let seg=0; seg<detail; ++seg){
        let f0 = seg/detail, f1 = (seg+1)/detail;
        let t0 = f0*2*Math.PI, t1 = f1*2*Math.PI;
        let colIdx = (arms*a/detail + f0 + (time*0.065)+repeat*0.12)%curPalette.length;
        let colA = curPalette[Math.floor(colIdx)%curPalette.length];
        let colB = curPalette[Math.floor(colIdx+1)%curPalette.length];
        let color = lerpColor(colA, colB, colIdx%1);
        ctx.strokeStyle = color;
        ctx.beginPath();
        let r0 = 185 + depth*0.15*Math.sin(t0*3 + time*0.7+repeat) + 35*Math.sin(a+evolve) + mandalaWave(t0, time+repeat*0.93, a)*depth * (0.88-0.26*repeat);
        let r1 = 185 + depth*0.15*Math.sin(t1*3 + time*0.7+repeat) + 35*Math.sin(a+evolve) + mandalaWave(t1, time+repeat*0.93, a)*depth * (0.88-0.26*repeat);
        let x0 = Math.cos(t0)*r0, y0 = Math.sin(t0)*r0;
        let x1 = Math.cos(t1)*r1, y1 = Math.sin(t1)*r1;
        ctx.moveTo(x0, y0);
        ctx.lineTo(x1, y1);
        ctx.stroke();
      }
      ctx.restore();
    }
    ctx.restore();
  }
  ctx.globalAlpha = 1.0;

  // Center glow
  let radial = ctx.createRadialGradient(0,0,0,0,0,70+depth*0.5);
  radial.addColorStop(0, lerpColor(curPalette[0], curPalette[curPalette.length-1], 0.3));
  radial.addColorStop(1, "#0000");
  ctx.save();
  ctx.translate(centerX, centerY);
  ctx.fillStyle = radial;
  ctx.beginPath();
  ctx.arc(0,0,70+depth*0.5,0,2*Math.PI);
  ctx.fill();
  ctx.restore();

  ctx.restore();
}

let lastT = 0;
function animate(ts){
  let time = (ts/1000) * animSpeed;
  drawMandala(time);
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);

// Controls
const armsControl = document.getElementById('armsControl');
const armsVal = document.getElementById('armsVal');
armsControl.addEventListener('input', e => {
  arms = parseInt(e.target.value);
  armsVal.textContent = arms;
});

const detailControl = document.getElementById('detailControl');
const detailVal = document.getElementById('detailVal');
detailControl.addEventListener('input', e => {
  detail = parseInt(e.target.value);
  detailVal.textContent = detail;
});

const depthControl = document.getElementById('depthControl');
const depthVal = document.getElementById('depthVal');
depthControl.addEventListener('input', e => {
  depth = parseInt(e.target.value);
  depthVal.textContent = depth;
});

const complexityControl = document.getElementById('complexityControl');
const complexityVal = document.getElementById('complexityVal');
complexityControl.addEventListener('input', e => {
  complexity = parseInt(e.target.value);
  complexityVal.textContent = complexity;
});

const speedControl = document.getElementById('speedControl');
const speedVal = document.getElementById('speedVal');
speedControl.addEventListener('input', e => {
  animSpeed = parseInt(e.target.value)/25;
  speedVal.textContent = animSpeed.toFixed(1) + "x";
});

const paletteSelector = document.getElementById('paletteSelector');
paletteSelector.addEventListener('change', e => {
  curPalette = palettes[e.target.value];
});

const bgColorPicker = document.getElementById('bgColorPicker');
bgColorPicker.addEventListener('input', e => {
  bgColor = e.target.value;
});
</script>
