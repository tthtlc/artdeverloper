---
layout: fullscreen
title: "Psychedelic Radial Wave Bloom"
tags:
  - graphics
---

<canvas id="bloomCanvas" width="700" height="700" style="display:block; margin:auto; background:#101022"></canvas>
<script>
const canvas = document.getElementById('bloomCanvas');
const ctx = canvas.getContext('2d');
const w = canvas.width, h = canvas.height;
const cx = w/2, cy = h/2;

// --- PARAMS ---
const petalCount = 14;
const petalBaseLen = 170;
const layerCount = 8;
const timeMultiplier = 0.35;
const oscMult = 0.61;
const globalRotationSpeed = 0.027;
const bgAlpha = 0.2; // motion trails

// PALETTE (psychedelic)
const palette = [
  '#f9fc53', // acid yellow
  '#f360db', // pink
  '#43ffeb', // cyan
  '#00ff8f', // lime-teal
  '#fd774b', // orange
  '#f0f',    // magenta
  '#12c2e9', // blue
  '#fdffb6'  // pale yellow
];

// Helper: get oscillating "wave" radius for a petal
function waveRadius(angle, layer, t) {
  // Each layer has different wave amplitude and phase
  const base = petalBaseLen + 24*layer;
  const amp = 33 + 19*Math.sin(layer+1.3);
  const freq = 2.2 + 0.3*layer;
  return base + amp * Math.sin(freq*angle + oscMult*layer + t*0.9 + 1.1*layer);
}

// Helper: HSLA string from [h,s,l,a]
function hsla(h,s,l,a) { return `hsla(${h},${s}%,${l}%,${a})`; }

// Animation main loop
function draw(t) {
  // BG fading for trails
  ctx.globalAlpha = bgAlpha;
  ctx.fillStyle = '#101022';
  ctx.fillRect(0,0,w,h);
  ctx.globalAlpha = 1;

  // global rotation
  const baseRot = t*globalRotationSpeed + 0.7*Math.sin(t*0.11);

  // "Glow pulse"
  const pulse = 0.34 + 0.3*Math.sin(t*0.37);

  // Draw layers from outside to inside
  for(let layer = layerCount-1; layer >= 0; layer--) {
    // Layer alpha fades to center
    let la = 0.7*(1-layer/layerCount) + 0.15*pulse;
    if(layer%2===0) la += 0.16; // boost even layers
    // Color
    let col = palette[layer % palette.length];
    let glow = layer > 0 ? 24 + 12*layer : 0;
    ctx.save();
    ctx.translate(cx, cy);
    ctx.rotate(baseRot * (1 + 0.08*layer));
    ctx.shadowColor = col;
    ctx.shadowBlur = glow * (1+pulse*0.9);

    // Petals in this layer
    for(let p=0; p < petalCount; p++) {
      let ang = (2*Math.PI/petalCount)*p; // base angle
      ang += Math.sin(t*0.11 + 0.45*p + layer)*0.12*layer; // undulate

      ctx.save();
      ctx.rotate(ang);
      // Wavy edge: build outline points
      ctx.beginPath();
      const steps = 28 + layer*2;
      for(let i=0; i<=steps; i++) {
        const a = Math.PI*((i/steps)-0.5); // petal span
        const wopt = t*timeMultiplier + 0.27*p + layer*0.52;
        let r = waveRadius(a,layer,wopt);
        // Add organic edge wiggle
        r += 11*Math.sin(4*a + wopt*0.8 + 0.3*layer + Math.cos(a*3 + t*0.22));
        if(i===0) ctx.moveTo(r,0);
        else ctx.lineTo(r*Math.cos(a), r*Math.sin(a));
      }
      // Petal tip curve
      for(let i=steps; i>=0; i--) {
        const a = Math.PI*((i/steps)-0.5);
        let tipR = waveRadius(-a,layer,wopt);
        tipR -= 0.63*petalBaseLen; // inward curve for closing
        ctx.lineTo(tipR*Math.cos(-a), tipR*Math.sin(-a));
      }
      ctx.closePath();

      // Color variation per petal
      if(layer%2===0) {
        ctx.fillStyle = col+"77";
      } else {
        // Hue rotate for odd layers
        const h = (360*(p/petalCount) + 40*layer + 210)%360;
        ctx.fillStyle = hsla(h, 84, 58, la);
      }
      ctx.strokeStyle = col + "cc";
      ctx.lineWidth = 1.3 + 0.5*layer;

      ctx.globalAlpha = la;
      ctx.fill();
      ctx.stroke();
      ctx.globalAlpha = 1;
      ctx.restore();
    }
    ctx.shadowBlur = 0;
    ctx.restore();
  }

  // Central core rotating eye
  let glowCore = 19 + 22*pulse;
  ctx.save();
  ctx.translate(cx, cy);
  ctx.rotate(baseRot*2.5);
  ctx.shadowColor = '#fff';
  ctx.shadowBlur = glowCore;
  ctx.beginPath();
  ctx.arc(0,0,32+7*Math.sin(t*0.91), 0, 2*Math.PI);
  ctx.fillStyle = hsla(300+60*Math.sin(t*0.27), 70, 74, 0.66+0.2*pulse);
  ctx.fill();
  ctx.shadowBlur = 0;
  ctx.beginPath();
  ctx.arc(0,0,10.5+5*pulse, 0, 2*Math.PI);
  ctx.fillStyle = "#101022";
  ctx.fill();
  ctx.beginPath();
  ctx.arc(3.1+4*Math.sin(t*0.75),-2.2+4*Math.cos(t*0.54),3+2*pulse, 0, 2*Math.PI);
  ctx.fillStyle = "#c3fd22";
  ctx.globalAlpha = 0.85;
  ctx.fill();
  ctx.restore();
}

// Animate
let start = undefined;
function animate(ts) {
  if(!start) start = ts;
  let t = (ts-start)/1000;
  draw(t);
  requestAnimationFrame(animate);
}

animate();
</script>
