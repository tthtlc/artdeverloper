---
layout: fullscreen
title: "Psychedelic Sinewave Orbital Swarm"
tags:
  - graphics
---

<canvas id="psy_swarm" width="700" height="700"></canvas>
<script>
/**
 * Psychedelic Sinewave Orbital Swarm
 * - Orbits of colored particles swarming around phasing sinewave orbits
 * - Color-cycling trails, evolving harmonically in time
 * - Self-contained, no dependencies
 */

const canvas = document.getElementById('psy_swarm');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const CX = W/2, CY = H/2;

const NUM_ORBS = 38;
const HARMONICS = 4;
const BASE_RADIUS = 220;
const ORB_TRAIL_LEN = 38;
const SPEED = 0.016;

// Create harmonic parameters for orbits
let harmonics = [];
for(let i=0; i<HARMONICS; i++){
  harmonics.push({
    amp: BASE_RADIUS * (0.46 + 0.4 * Math.random()),
    freq: 0.57 + 1.8 * Math.random(),
    phase: Math.random() * Math.PI * 2
  });
}

// Each orb follows a unique evolving multi-harmonic path
function makeOrb(k) {
  const hue0 = 360 * k / NUM_ORBS;
  return {
    t: Math.random()*Math.PI*2,
    angSpeed: 0.015 + 0.025*Math.random(),
    colorSeed: hue0,
    trail: [],
    noise: 0.17 + Math.random()*0.15
  }
}
let orbs = [];
for(let i=0; i<NUM_ORBS; i++) {
  orbs.push(makeOrb(i));
}

// Helper for crazy color
function rainbowColor(angle, brightness=0.65) {
  // angle: 0..2pi
  let h = ((angle/Math.PI/2)*360)%360;
  let l = 52 + 36*Math.sin(angle*0.7);
  return `hsl(${h.toFixed(1)},98%,${l*brightness}%)`;
}

// Animation loop
let t = 0;
function animate() {
  t += SPEED;

  // Fade previous trails
  ctx.globalAlpha = 0.19;
  ctx.fillStyle = '#000013';
  ctx.fillRect(0,0,W,H);
  ctx.globalAlpha = 1.0;

  // For each orb
  orbs.forEach((orb, i) => {
    // Compose multiple sinewave harmonics for evolving orbital radius and angle
    let theta = orb.t + t*orb.angSpeed * (1.7 + Math.sin(0.8*t+orb.colorSeed));
    let r = 0, offset = 0;
    for(let j=0; j<HARMONICS; j++) {
      r += harmonics[j].amp * Math.sin(theta*harmonics[j].freq + harmonics[j].phase + offset);
      offset += 5.1; //differentiate phasing
    }
    r /= HARMONICS;
    r += BASE_RADIUS * 0.2;
    // Sinusoidal flower-pattern modulation
    r += Math.sin(theta*2 + Math.sin(t+orb.colorSeed)) * 52;

    // Position
    let phi = theta + Math.sin(theta + t*0.2) * orb.noise * 2;
    let x = CX + r * Math.cos(phi);
    let y = CY + r * Math.sin(phi);

    // Trail update
    orb.trail.push({x, y, time:t});
    if(orb.trail.length>ORB_TRAIL_LEN) orb.trail.shift();

    // Draw trail (older = more transparent)
    for(let ti=0; ti<orb.trail.length-1; ti++){
      let ptA = orb.trail[ti], ptB = orb.trail[ti+1];
      let k = (ti+1)/orb.trail.length;
      ctx.strokeStyle = rainbowColor(orb.colorSeed + ptB.time*1.4 + i*0.12, k);
      ctx.lineWidth = 2.3-1.3*k + 1.5*Math.sin(orb.colorSeed/19 + t*0.9);
      ctx.globalAlpha = 0.19+k*0.87;
      ctx.beginPath();
      ctx.moveTo(ptA.x, ptA.y);
      ctx.lineTo(ptB.x, ptB.y);
      ctx.stroke();
    }
    ctx.globalAlpha = 1.0;

    // Glowy orb head
    ctx.save();
    ctx.beginPath();
    ctx.arc(x, y, 7.5+2.7*Math.sin(t*1.5+orb.colorSeed), 0, Math.PI*2);
    // Animate the head color with orbital progress
    ctx.strokeStyle = rainbowColor(theta*2 + orb.colorSeed + t*0.3, 1.05);
    ctx.shadowBlur = 22;
    ctx.shadowColor = ctx.strokeStyle;
    ctx.lineWidth = 2.4;
    ctx.globalAlpha = 0.96;
    ctx.stroke();
    ctx.globalAlpha = 0.44;
    ctx.fillStyle = ctx.strokeStyle;
    ctx.fill();
    ctx.restore();
    ctx.globalAlpha = 1.0;

    // Advance time parameter
    orb.t += orb.angSpeed * (1 + Math.sin(0.19*t+i*0.12));
  });

  // Central oscillating psychedelic "sun" pulse
  ctx.save();
  let srad = 38 + 22*Math.sin(t*0.8);
  let rgrad = ctx.createRadialGradient(CX, CY, 0, CX, CY, srad*2.2);
  rgrad.addColorStop(0.00, 'rgba(255,255,210,0.17)');
  rgrad.addColorStop(0.28, rainbowColor(t*2.6, 1.14));
  rgrad.addColorStop(0.62, rainbowColor(-t*2.7+2.2,0.8));
  rgrad.addColorStop(1.0, 'rgba(40,20,55,0.0)');
  ctx.globalAlpha=0.92;
  ctx.beginPath();
  ctx.arc(CX, CY, srad*2.2, 0, Math.PI*2);
  ctx.fillStyle=rgrad;
  ctx.fill();
  ctx.restore();

  requestAnimationFrame(animate);
}

// Initial fade fill
ctx.fillStyle = "#000013";
ctx.fillRect(0,0,W,H);

animate();
</script>
