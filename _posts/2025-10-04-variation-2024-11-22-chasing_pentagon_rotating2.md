---
layout: fullscreen
title: "Psychedelic Radiant Wave Circles"
tags:
  - graphics
---

<canvas id="radiantCircleCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('radiantCircleCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width;
const H = canvas.height;

// Color palette: psychedelic rainbow
function getColor(t) {
  // t in [0, 1]
  let r = Math.floor(128 + 127 * Math.sin(2 * Math.PI * t + 0));
  let g = Math.floor(128 + 127 * Math.sin(2 * Math.PI * t + 2));
  let b = Math.floor(128 + 127 * Math.sin(2 * Math.PI * t + 4));
  return `rgb(${r},${g},${b})`;
}

// Parameters for concentric modulated circles
const numCircles = 18;
const baseRadius = 58;
const radiusIncrement = 18;
const waveAmplitude = 34;
const waveFreqs = [5, 8, 3, 13];

// "Particle" system parameters (moving points along circumference)
const numParticles = 42;

// Main animation function
let time = 0;
function draw() {
  ctx.globalAlpha = 1;
  ctx.globalCompositeOperation = 'source-over';
  ctx.clearRect(0, 0, W, H);

  // Draw glowing black background
  let grad = ctx.createRadialGradient(W/2, H/2, 135, W/2, H/2, W/2);
  grad.addColorStop(0, "#181021");
  grad.addColorStop(0.8, "#000004");
  ctx.fillStyle = grad;
  ctx.fillRect(0, 0, W, H);

  // Draw concentric modulated "wavy" circles with rainbow gradient
  for (let k=0; k<numCircles; k++) {
    ctx.save();
    ctx.translate(W/2, H/2);
    ctx.beginPath();
    let waveMult = 0.9 + 0.2*Math.sin(time*0.6 + k*0.7);

    for (let i=0; i<=200; i++) {
      let t = i/200;
      let theta = t * 2 * Math.PI;
      let mod = 0;
      for (let f=0; f<waveFreqs.length; f++) {
        let A = waveAmplitude/(f+2) * waveMult;
        let freq = waveFreqs[f] + k*0.08 + Math.sin(time*0.33+f*1.74)*0.6;
        let phase = time * (0.09 + 0.13*k + 0.07*f) + f*3 + k*0.9;
        mod += A * Math.sin(freq*theta + phase);
      }
      let R = baseRadius + radiusIncrement*k + mod;
      let x = R * Math.cos(theta);
      let y = R * Math.sin(theta);

      // Color ribbon
      ctx.strokeStyle = getColor( (t + 0.13*k + 0.11*time) % 1 );
      ctx.lineWidth = 1.3 + Math.sin(time + k)*0.3;

      if (i===0)
        ctx.moveTo(x, y);
      else
        ctx.lineTo(x, y);
    }
    ctx.shadowColor = getColor((k*0.08 + 0.003*time)%1);
    ctx.shadowBlur = 13 + 3*Math.sin(time/2 + k*0.7);
    ctx.stroke();
    ctx.restore();
  }

  // Draw radiant moving particles on outer rim with trail
  ctx.globalCompositeOperation = 'lighter';
  for(let p=0; p<numParticles; p++) {
    let phase = (p/numParticles)*2*Math.PI + Math.sin(time*0.3+p)*0.18;
    let pr = baseRadius + radiusIncrement*(numCircles-1) + waveAmplitude*0.75;
    let pspeed = 0.9 + 0.45*Math.sin(time*0.13 + p*0.7);
    let theta = phase + time*pspeed*0.18 + Math.sin(time+p)*0.09;
    let x = W/2 + pr * Math.cos(theta + Math.sin(time*0.79+p)*0.15);
    let y = H/2 + pr * Math.sin(theta + Math.cos(time*0.79-p)*0.13);

    // Trailing afterimages
    for(let tTrail=4; tTrail>=0; tTrail--) {
      let tt = time - tTrail*0.089;
      let x2 = W/2 + pr * Math.cos(phase + tt*pspeed*0.18 + Math.sin(tt+p)*0.09 + Math.sin(tt*0.79+p)*0.15);
      let y2 = H/2 + pr * Math.sin(phase + tt*pspeed*0.18 + Math.cos(tt*0.79-p)*0.13);
      ctx.beginPath();
      ctx.arc(x2, y2, 6 - tTrail, 0, 2*Math.PI);
      ctx.fillStyle = getColor( ((p/numParticles)+(tt*0.04))%1 );
      ctx.globalAlpha = 0.12 + 0.22/(1+tTrail);
      ctx.shadowColor = getColor(((p/numParticles)+0.13+tt*0.09)%1);
      ctx.shadowBlur = 14-2*tTrail;
      ctx.fill();
    }
  }
  ctx.globalAlpha = 1;
  time += 0.0325;
  requestAnimationFrame(draw);
}

draw();
</script>
