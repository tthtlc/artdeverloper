---
layout: fullscreen
title: "Fractal Pulsar Webs: Psychedelic Particle Blooms"
tags:
  - graphics
---

<style>
/* Hide Jekyll layout elements for fullscreen effect */
.navbar, .intro-header, .post-container, .sidebar-container, footer {
  display: none !important;
}

body {
  margin: 0;
  padding: 0;
  background: #090f24;
  overflow: hidden;
  min-height: 100vh;
}

canvas {
  display: block;
  width: 100vw;
  height: 100vh;
  cursor: pointer;
  background: radial-gradient(circle at 55% 58%, #111038 0%, #0c0512 100%);
  position: fixed;
  top: 0;
  left: 0;
  z-index: 9999;
}
</style>

<canvas id="pulsar-web"></canvas>

<script>
const canvas = document.getElementById('pulsar-web');
const ctx = canvas.getContext('2d');
let width, height, dpr = window.devicePixelRatio || 1;

function resize() {
  width = window.innerWidth;
  height = window.innerHeight;
  canvas.width = width * dpr;
  canvas.height = height * dpr;
  ctx.setTransform(1, 0, 0, 1, 0, 0);
  ctx.scale(dpr, dpr);
}
window.addEventListener('resize', resize);
resize();

// Helper: HSV to RGB
function hsv2rgb(h, s, v){
  let f = (n, k = (n + h/60)%6) => v - v*s*Math.max(Math.min(k, 4 - k, 1), 0);
  return [f(5),f(3),f(1)];
}

// Particle system parameters
const PARTICLE_GROUPS = 7;
const PARTICLES_PER_GROUP = 48;
const WEB_ARMS = 11;
const CENTRAL_RADII = [0.2, 0.38, 0.58, 0.8];

let t0 = performance.now();

function draw(now) {
  const t = (now-t0)*0.00018;
  ctx.globalCompositeOperation = "lighter";
  ctx.clearRect(0, 0, width, height);

  // Soft glowing fractal webs
  for(let r = 0; r < CENTRAL_RADII.length; ++r) {
    let baseHue = ((t*40 + r*75) % 360);
    let webRadius = CENTRAL_RADII[r] * Math.min(width, height);
    let pulse = 0.91 + 0.08 * Math.sin(t*2.6 + r * Math.PI * 0.9);
    for(let arm=0; arm<WEB_ARMS; ++arm){
      let armAng = (2*Math.PI / WEB_ARMS) * arm + Math.sin(t*0.8 + arm*1.2) * 0.11;
      let alpha = 0.044 + 0.044*Math.abs(Math.sin(t*2 + arm+10*r));
      ctx.save();
      ctx.beginPath();
      for(let i=0; i<=22; ++i){
        let ang = armAng + i*0.098 + Math.sin(i*0.21 + t)*0.009;
        let wobble = Math.sin((ang*6) + t*1.3 + r*3.7 + arm*1.8) * 0.2 + Math.cos(ang*1.5 + t*1.9) * 0.1;
        let rad = webRadius * pulse * (1 + wobble*0.17);
        let x = width*0.5 + Math.cos(ang+t*0.16+r*0.9) * rad;
        let y = height*0.5 + Math.sin(ang-t*0.22-arm*0.12) * rad;
        if(i===0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      let rgb = hsv2rgb((baseHue+arm*360/WEB_ARMS)%360, 0.95, 0.75 + 0.12*Math.sin(t*2+arm+r*4));
      ctx.shadowBlur = 16 + 9*r + 8*Math.abs(Math.sin(t+r*1.8 + arm*3));
      ctx.shadowColor = `rgba(${(rgb[0]*180).toFixed()}, ${(rgb[1]*170).toFixed()}, ${(rgb[2]*255).toFixed()},0.73)`;
      ctx.strokeStyle = `rgba(${(rgb[0]*255).toFixed()}, ${(rgb[1]*255).toFixed()}, ${(rgb[2]*255).toFixed()},${alpha})`;
      ctx.lineWidth = 2.7 + 1.0*Math.abs(Math.sin(t*2+arm*0.9));
      ctx.stroke();
      ctx.restore();
    }
  }

  // Pulsating, swirling particle blooms
  for(let g=0; g<PARTICLE_GROUPS; ++g){
    let hue = ((t*80 + g*60) % 360);
    let spiralFreq = 0.21 + 0.11*Math.cos(t+g*1.2);
    let orbitR = Math.min(width,height)* (0.18 + 0.19*Math.sin(t*1.1+g));
    for(let p=0; p<PARTICLES_PER_GROUP; ++p) {
      let phi = 2*Math.PI*p/PARTICLES_PER_GROUP + Math.sin(t*0.6 + p*0.3 + g);
      let spiralT = t*2 + g*0.8 - p*spiralFreq;
      let nx = Math.cos(phi+spiralT) * (orbitR + 12*sinT(p, g, t)) + width/2;
      let ny = Math.sin(phi-spiralT*0.8) * (orbitR + 10*cosT(p, g, t)) + height/2;
      let sz = 4.6 + 1.6*Math.sin(t*1.5+p*0.21+g);
      let partHue = hue + p*3 + 100*Math.sin(p*0.2+t*0.41+g*2);
      let rgb = hsv2rgb(partHue%360, 0.97, 0.84 + 0.18*Math.cos(g+p+t));
      ctx.save();
      ctx.beginPath();
      ctx.arc(nx, ny, sz, 0, Math.PI*2);
      ctx.globalAlpha = 0.45 + 0.16*Math.sin(t+p*0.5+g*0.9);
      ctx.shadowBlur = 18 + 10*Math.abs(Math.sin(t+g+p*0.6));
      ctx.shadowColor = `rgba(${(rgb[0]*160).toFixed()}, ${(rgb[1]*190).toFixed()}, ${(rgb[2]*255).toFixed()},0.56)`;
      ctx.fillStyle = `rgba(${(rgb[0]*255).toFixed()}, ${(rgb[1]*255).toFixed()}, ${(rgb[2]*255).toFixed()}, 1)`;
      ctx.fill();
      ctx.restore();
    }
  }

  // Central fractal bloom (kaleidoscopic polygon-pulse)
  let cx = width/2, cy = height/2;
  let polySides = 6 + Math.floor(2+3*Math.abs(Math.sin(t*1.8)));
  let polyAng = Math.PI/2 + t*2.2;
  let radiusPulse = 52 + 16*Math.sin(t*3.5);
  ctx.save();
  ctx.beginPath();
  for(let i=0; i<=polySides; ++i){
    let ang = polyAng + i*2*Math.PI/polySides;
    let r = radiusPulse*(1 + 0.15*Math.sin(i*5.1+t+i));
    let x = cx + Math.cos(ang)*r;
    let y = cy + Math.sin(ang)*r;
    if(i===0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
  }
  let hcv = ((t*160) % 360);
  let rgb = hsv2rgb(hcv, 0.94, 0.99);
  ctx.globalAlpha = 0.29 + 0.15*Math.sin(t*4.7);
  ctx.shadowBlur = 36 + 16*Math.abs(Math.sin(t*2.3));
  ctx.shadowColor = `rgba(${(rgb[0]*240).toFixed()}, ${(rgb[1]*210).toFixed()}, ${(rgb[2]*255).toFixed()},0.81)`;
  ctx.fillStyle = `rgba(${(rgb[0]*255).toFixed()}, ${(rgb[1]*255).toFixed()}, ${(rgb[2]*255).toFixed()}, 1)`;
  ctx.fill();
  ctx.restore();

  // Subtle dreamy overlay
  ctx.globalAlpha = 0.057;
  ctx.globalCompositeOperation = "lighter";
  ctx.fillStyle = `rgba(14,8,22,0.06)`;
  ctx.fillRect(0,0,width,height);

  requestAnimationFrame(draw);
}

// Helper oscillators for orbits
function sinT(p, g, t) {
  return Math.sin(t*1.6 + p*0.25 + g)*18 + Math.cos(p*0.11 - t*1.8 + g*0.6)*7;
}
function cosT(p, g, t) {
  return Math.cos(t*2.1 + p*0.29 + g*0.45)*15 + Math.sin(p*0.09+t*2)*4;
}

// Interactive: mouse click to "pulse" the spectrum
canvas.addEventListener('click', e => {
  t0 -= 400 + 400*Math.random();
});

draw(t0);
</script>