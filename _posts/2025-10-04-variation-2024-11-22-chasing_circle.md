---
layout: fullscreen
title: "Psychedelic Ripple Flowers: Generative Waves in a Circle"
tags:
  - graphics
---

A hypnotic ripple flower made of many undulating, color-shifting petals. The waveforms, colors, and shapes are modulated by interactive controls, creating constantly evolving, psychedelic motion. The animation is fully procedural and reacts instantly to parameter changes below.

<canvas id="rippleFlowerCanvas" width="800" height="800" style="background:#101323; display:block; margin: 0 auto; border-radius:15px; box-shadow:0 0 64px #405 0;"></canvas>
<div class="controls">
  <div class="control-container">
    <label for="petals">Petals (waves): </label>
    <input type="range" id="petals" min="3" max="32" value="12">
    <span id="petalsValue">12</span>
  </div>
  <div class="control-container">
    <label for="amplitude">Petal Amplitude: </label>
    <input type="range" id="amplitude" min="10" max="180" value="90">
    <span id="amplitudeValue">90</span>
  </div>
  <div class="control-container">
    <label for="colorCycle">Color Cycle Speed: </label>
    <input type="range" id="colorCycle" min="0" max="180" value="60">
    <span id="colorCycleValue">60</span>
  </div>
  <div class="control-container">
    <label for="waveFreq">Inner Wave Frequency: </label>
    <input type="range" id="waveFreq" min="1" max="12" value="4">
    <span id="waveFreqValue">4</span>
  </div>
</div>

<script>
const canvas = document.getElementById("rippleFlowerCanvas");
const ctx = canvas.getContext("2d");

const centerX = canvas.width/2;
const centerY = canvas.height/2;
const baseRadius = 220;

const petalsSlider = document.getElementById('petals');
const amplitudeSlider = document.getElementById('amplitude');
const colorCycleSlider = document.getElementById('colorCycle');
const waveFreqSlider = document.getElementById('waveFreq');

const petalsValue = document.getElementById('petalsValue');
const amplitudeValue = document.getElementById('amplitudeValue');
const colorCycleValue = document.getElementById('colorCycleValue');
const waveFreqValue = document.getElementById('waveFreqValue');

function updateLabels() {
  petalsValue.textContent = petalsSlider.value;
  amplitudeValue.textContent = amplitudeSlider.value;
  colorCycleValue.textContent = colorCycleSlider.value;
  waveFreqValue.textContent = waveFreqSlider.value;
}
updateLabels();

petalsSlider.addEventListener('input', updateLabels);
amplitudeSlider.addEventListener('input', updateLabels);
colorCycleSlider.addEventListener('input', updateLabels);
waveFreqSlider.addEventListener('input', updateLabels);

// Main animation function
let lastPetals = -1;
let shapePath = [];

function draw(time) {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Parameters
  const petals = parseInt(petalsSlider.value);
  const amplitude = parseFloat(amplitudeSlider.value);
  const colorCycle = parseFloat(colorCycleSlider.value);
  const waveFreq = parseInt(waveFreqSlider.value);

  // Precompute the shape path if number of points doesn't change
  const points = 600;

  // Animate base rotation and wave phase
  const t = time * 0.001;
  const angleOffset = t * 0.14;
  const wavePhase = t * 0.4;
  const colorPhase = t * (0.02 + colorCycle * 0.0025);

  if (petals !== lastPetals)
    shapePath = [];
  lastPetals = petals;

  // Draw each layer (petal body, outlines, aura)
  for (let layer=3; layer>=0; layer--) {
    let rMod = 1 - layer*0.07;
    let alpha = [0.20, 0.35, 0.5, 1.0][layer];
    let lw = [0, 0, 1.5, 2.5][layer];

    ctx.save();
    ctx.globalAlpha = alpha;
    if (lw > 0) ctx.lineWidth = lw;
    else ctx.lineWidth = 14-6*layer;

    let gradient = ctx.createRadialGradient(centerX, centerY, baseRadius*0.6, centerX, centerY, baseRadius*1.05);
    gradient.addColorStop(0, 'rgba(48,50,80,0.8)');
    gradient.addColorStop(1, 'rgba(0,0,16,0.3)');
    ctx.strokeStyle = layer<3 ? gradient : "#fff";

    ctx.beginPath();
    for (let i=0;i<=points;i++) {
      // Angular position
      const theta = (2 * Math.PI * i/points) + angleOffset;

      // The rippled flower/petal radius
      const wave =
         Math.cos(theta * petals + wavePhase*2 + Math.sin(theta*waveFreq + wavePhase)*1.4)*0.5
        + Math.sin(theta*petals*0.5+wavePhase*1.2) * 0.3
        + Math.sin(theta*waveFreq + wavePhase*1.3) * 0.15
        ;
      const r =
        baseRadius * rMod +
        amplitude * wave;

      // Calculate position
      const x = centerX + r * Math.cos(theta);
      const y = centerY + r * Math.sin(theta);

      if (i === 0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);

      // Cache for filling inner flower
      if (layer === 3) {
        if (!shapePath[i]) shapePath[i] = {};
        shapePath[i].x = x;
        shapePath[i].y = y;
      }
    }
    ctx.closePath();

    if (layer === 3) {
      // Fill inner flower with psychedelic radial gradient stripes
      let petColorBase = (colorPhase * 65) % 360;
      let petalSteps = petals*2 + 3;
      ctx.save();
      ctx.clip();
      for(let p=0; p<petalSteps; ++p) {
        let c1 = "hsl(" + ((petColorBase + p*360/petalSteps)%360) + ",92%,60%)";
        let c2 = "hsl(" + ((petColorBase + (p+1)*360/petalSteps+32)%360) + ",94%,45%)";
        ctx.beginPath();
        for(let j=0;j<=points;++j) {
          let theta = (2 * Math.PI * j / points) + angleOffset;
          if(Math.floor(j*petalSteps/points) === p) {
            let x = shapePath[j].x;
            let y = shapePath[j].y;
            if (j===0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
          }
        }
        ctx.closePath();

        let gradPetal = ctx.createRadialGradient(
           centerX + Math.cos(2*Math.PI*p/petalSteps)*baseRadius*0.16,
           centerY + Math.sin(2*Math.PI*p/petalSteps)*baseRadius*0.16,
           baseRadius*0.11,
           centerX, centerY, baseRadius*1.2
        );
        gradPetal.addColorStop(0, c1);
        gradPetal.addColorStop(0.95, c2);
        gradPetal.addColorStop(1, 'rgba(0,0,32,0)');
        ctx.fillStyle = gradPetal;
        ctx.globalAlpha = 0.94;
        ctx.fill();
      }
      ctx.restore();
    } else {
      ctx.stroke();
    }
    ctx.restore();
  }

  // Sparkles and radiating points (pupil dilation!)
  let nSparks = 36;
  for (let i = 0; i < nSparks; ++i) {
    let angle = 2 * Math.PI * i / nSparks + angleOffset*1.6;
    let r = baseRadius*1.1 + Math.sin(t*4.7+angle*3)*12 + Math.sin(t*3.1+angle*2.2)*7;
    let x = centerX + r * Math.cos(angle);
    let y = centerY + r * Math.sin(angle);
    let hue = (colorPhase*290 + i*29)%360;
    ctx.save();
    ctx.globalAlpha = 0.26 + 0.14*Math.sin(t*2+angle*8);
    ctx.beginPath();
    ctx.arc(x, y, 3 + 2*Math.pow(Math.abs(Math.sin(t*5.5+angle)),1.4), 0, 2 * Math.PI);
    ctx.fillStyle = `hsl(${hue},100%,70%)`;
    ctx.shadowColor = `hsl(${hue},98%,89%)`;
    ctx.shadowBlur = 6;
    ctx.fill();
    ctx.restore();
  }

  requestAnimationFrame(draw);
}
requestAnimationFrame(draw);
</script>

<style>
.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 18px;
  justify-content: center;
  margin: 26px auto 0 auto;
  max-width: 650px;
}
.control-container {
  background: #141428;
  border-radius: 9px;
  box-shadow: 0 1px 12px #0003, 0 0 0 1px #37244a44;
  padding: 8px 20px;
  margin: 6px 0;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
label {
  color: #e0d0fe;
  font-weight: bold;
  font-size: 1em;
}
input[type="range"] {
  width: 190px;
  accent-color: #9f65ff;
}
span {
  color: #77f8de;
  font-weight: 500;
  font-family: monospace;
  margin-left: 4px;
  font-size: 1.05em;
}
@media (max-width: 800px) {
  #rippleFlowerCanvas { width:96vw !important; height:96vw !important; max-width:100vw; max-height:100vw;}
  .controls { max-width: 99vw; }
}
</style>
