---
layout: fullscreen
title: Psychedelic Hypnotic Waves with Chromatic Orbits
tags:
  - graphics
---

  <style>
    #container {
      position: relative;
      width: 500px;
      height: 500px;
      margin: auto;
    }
    canvas {
      position: absolute;
      top: 0;
      left: 0;
      border: 1px solid #070707;
      display: block;
      background: #181a33;
    }
    #controls {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 20px;
    }
    #controls label {
      margin: 10px 0;
      color: #fff;
      font-family: monospace;
    }
    .canvas-container {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
    .footer-link { 
      position: absolute; 
      bottom: 20px; 
      text-align: center; 
      width: 100%; 
    } 
    .footer-link a { 
      font-size: 16px; 
      color: #b7e9ff; 
      text-decoration: none; 
    } 
    .footer-link a:hover { 
      text-decoration: underline; 
    } 
</style>
  <div id="container">
    <canvas id="mainCanvas" width="500" height="500"></canvas>
    <canvas id="trailCanvas" width="500" height="500"></canvas>
  </div>
  <div id="controls">
    <label>
      Wave Count: <input type="range" id="waveCountSlider" min="2" max="12" value="6">
    </label>
    <label>
      Trail Fade: <input type="range" id="fadeSlider" min="0" max="100" value="8">
    </label>
    <label>
      Orbit Count: <input type="range" id="orbitCountSlider" min="1" max="8" value="3">
    </label>
  </div>
  <div class="canvas-container">
    <div id="gitalk-container"></div>
  </div>
<script>
document.addEventListener("contextmenu", e => e.preventDefault());

const mainCanvas = document.getElementById('mainCanvas');
const ctx = mainCanvas.getContext('2d');
const trailCanvas = document.getElementById('trailCanvas');
const trailCtx = trailCanvas.getContext('2d');

let width = mainCanvas.width, height = mainCanvas.height;
const centerX = width / 2, centerY = height / 2;

let waveCount = parseInt(document.getElementById('waveCountSlider').value);
let orbitCount = parseInt(document.getElementById('orbitCountSlider').value);
let fade = parseInt(document.getElementById('fadeSlider').value);

document.getElementById('waveCountSlider').addEventListener('input', function() {
  waveCount = parseInt(this.value);
  clearTrails();
});
document.getElementById('orbitCountSlider').addEventListener('input', function() {
  orbitCount = parseInt(this.value);
  clearTrails();
});
document.getElementById('fadeSlider').addEventListener('input', function() {
  fade = parseInt(this.value);
  // no need to clear trails, as fade will soft-reset
});

function clearTrails() {
  trailCtx.clearRect(0, 0, width, height);
}

function lerp(a,b,t) { return a + (b-a)*t; }
function hsvToRgb(h,s,v) {
  let f=(n,k=(n+h/60)%6)=>v-v*s*Math.max(Math.min(k,4-k,1),0);
  return [f(5)*255, f(3)*255, f(1)*255];
}

class OrbitingBall {
  constructor(orbitIndex, totalOrbits) {
    const theta = (orbitIndex / totalOrbits) * Math.PI * 2;
    this.baseRadius = lerp(80, 210, Math.abs(Math.sin(theta)));
    this.speed = lerp(0.01, 0.035, Math.random());
    this.phase = Math.random() * Math.PI * 2;
    this.colorOffset = theta * 180 / Math.PI;
    this.drift = lerp(0.7,1.3,Math.random());
  }
  position(t, mainAngle) {
    const r = this.baseRadius + 16 * Math.sin(mainAngle*1.22 + this.phase);
    const a = mainAngle*this.drift + this.phase;
    return {
      x: centerX + r * Math.cos(a),
      y: centerY + r * Math.sin(a)
    }
  }
  color(t) {
    const h = (t*60 + this.colorOffset)%360;
    const rgb = hsvToRgb(h, 0.85, 1);
    return `rgb(${rgb[0]|0},${rgb[1]|0},${rgb[2]|0})`;
  }
}

let orbits = [];

function updateOrbits() {
  orbits = [];
  for (let i=0; i<orbitCount; ++i) {
    orbits.push(new OrbitingBall(i, orbitCount));
  }
}
updateOrbits();

document.getElementById('orbitCountSlider').addEventListener('input', updateOrbits);

let t = 0;
function drawWavePattern(ctx, t, waveCount) {
  ctx.save();
  ctx.translate(centerX, centerY);

  ctx.beginPath();
  let pt = (ang) => {
    let w = waveCount;
    let baseR = lerp(170, 190, 0.5+0.35*Math.sin(t/44));
    let ampA = lerp(32,44,Math.sin(t/67));
    let ampB = lerp(14,23,Math.cos(t/39));
    let r = baseR 
      + ampA * Math.sin(w*ang - t/12) 
      + ampB * Math.sin((w+2)*ang + t/24)
      + 11 * Math.sin(0.7*ang + t/75);
    return [
      r * Math.cos(ang),
      r * Math.sin(ang)
    ];
  };
  // trail color: shifting rainbow based on t
  let rainbow = hsvToRgb((t*9)%360,0.88,0.95);
  ctx.strokeStyle = `rgba(${rainbow[0]|0},${rainbow[1]|0},${rainbow[2]|0},0.82)`;
  ctx.lineWidth = 2.4;
  for (let i=0; i<=200; ++i) {
    let ang = i/200 * 2 * Math.PI;
    let [x, y] = pt(ang);
    if (i==0) ctx.moveTo(x,y);
    else ctx.lineTo(x,y);
  }
  ctx.closePath();
  ctx.shadowColor = '#fff8';
  ctx.shadowBlur = 16;
  ctx.stroke();
  ctx.shadowBlur = 0;
  ctx.restore();
}

function draw() {
  ctx.clearRect(0,0,width,height);

  // Fade trail for motion blur style
  trailCtx.globalAlpha = lerp(0.82,0.95,fade/100);
  trailCtx.fillStyle = "#181a33";
  trailCtx.fillRect(0,0,width,height);
  trailCtx.globalAlpha = 1;

  // Draw undulating contour
  drawWavePattern(ctx, t, waveCount);

  // Draw orbits and balls with trails
  let mainAngle = t * 0.032;

  for (let i=0; i<orbits.length; ++i) {
    let orb = orbits[i];
    // Previous pos for trail
    let prev = orb.position(t-1, mainAngle-(orb.speed));
    // Current pos
    let curr = orb.position(t, mainAngle);

    // trail
    trailCtx.save();
    trailCtx.beginPath();
    trailCtx.moveTo(prev.x, prev.y);
    trailCtx.lineTo(curr.x, curr.y);
    trailCtx.strokeStyle = orb.color(t);
    trailCtx.lineWidth = 3.2;
    trailCtx.shadowColor = orb.color(t);
    trailCtx.shadowBlur = 10;
    trailCtx.globalAlpha = 0.96;
    trailCtx.stroke();
    trailCtx.shadowBlur = 0;
    trailCtx.restore();

    // Ball
    ctx.save();
    ctx.beginPath();
    ctx.arc(curr.x, curr.y, lerp(7.1,17,Math.abs(Math.sin(t/60+i))), 0, 2*Math.PI);
    ctx.fillStyle = orb.color(t);
    ctx.globalAlpha = 0.88;
    ctx.shadowColor = orb.color(t);
    ctx.shadowBlur = 15;
    ctx.fill();
    ctx.shadowBlur = 0;
    ctx.globalAlpha = 1.0;
    ctx.restore();
  }

  t += 1;
  requestAnimationFrame(draw);
}
draw();

</script>
<!-- Gitalk link  -->
<link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
<script src="https://unpkg.com/gitalk@latest/dist/gitalk.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/blueimp-md5/2.10.0/js/md5.min.js"></script>
<script type="text/javascript">
var gitalk = new Gitalk({
  clientID: 'Ov23lixOB0KjXtg08eAj',
  clientSecret: 'a3a33cad9733049a39849d54e99e69f70f69d1c1',
  repo: 'tthtlc.github.io',
  owner: 'tthtlc',
  admin: ['tthtlc'],
  distractionFreeMode: true,
  id: md5(location.pathname),
});
</script>
