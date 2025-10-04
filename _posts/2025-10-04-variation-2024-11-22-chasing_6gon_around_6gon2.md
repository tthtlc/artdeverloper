---
layout: fullscreen
title: "Hypnotic Pulsing Spiro-Waves"
tags:
  - graphics
---

<canvas id="spiroCanvas" width="800" height="800" style="display:block;margin:0 auto;background:#121030;"></canvas>
<script>
// --- CONFIG ---
const canvas = document.getElementById('spiroCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const CX = W/2, CY = H/2;

// --- PARAMETERS ---
const ARMS = 8;
const WAVES = 5;
const ARM_LENGTH = 270;
const BASE_RADIUS = 60;
const PARTICLES_PER_ARM = 110;
const COLOR_CYCLE_SPEED = 0.18;
const WAVE_SPEED = 0.045;
const WAVE_HEIGHT = 48;

// Helper: (for smooth color cycling)
function hsl(h, s, l, a=1) {
  return `hsla(${h%360},${s}%,${l}%,${a})`;
}

// Helper: ease in out sin
function easeSin(x) {
  return (1 - Math.cos(Math.PI * x)) / 2;
}

// Particle system
class ArmWave {
  constructor(angle, colorSeed) {
    this.angle = angle;
    this.colorSeed = colorSeed;
  }

  draw(t) {
    ctx.save();
    ctx.translate(CX, CY);
    ctx.rotate(this.angle);

    for (let i=0; i<PARTICLES_PER_ARM; i++) {
      // Progress along arm, normalized 0...1
      const progress = i/(PARTICLES_PER_ARM-1);
      // Radius modulated by base + multi-waves + breathing
      const baseR = BASE_RADIUS + ARM_LENGTH * progress;
      const oscillation = 
        Math.sin(WAVES * Math.PI * 2 * progress + t * WAVE_SPEED * 3 + this.colorSeed)
        * (WAVE_HEIGHT * (0.3 + 0.7*easeSin(progress)));
      const breathing = Math.sin(t*0.73 + baseR*0.018 + this.colorSeed) * 30 * easeSin(progress);

      const r = baseR + oscillation + breathing;

      // Spiral motion -- arms revolve gently over time
      const dynamicAngle = progress * 1.6*Math.PI 
        + Math.sin(t*0.29 + progress * 2.4 + this.colorSeed)*0.28
        + t*0.10
        ;

      const px = Math.cos(dynamicAngle) * r;
      const py = Math.sin(dynamicAngle) * r;

      // Color along arm (rainbow cycling)
      const hue = (this.colorSeed*60 + progress*280 + t*COLOR_CYCLE_SPEED*210) % 360;
      const sat = 80 - Math.sin(t*1.5+progress*16)*13;
      const lum = 45 + Math.sin(progress*8 + t*0.38 + this.colorSeed*2)*22;
      ctx.beginPath();
      ctx.arc(px, py, 2.1 + 1.2*Math.sin(t*1.7+progress*12 + this.colorSeed), 0, 2*Math.PI);
      ctx.fillStyle = hsl(hue, sat, lum);
      ctx.shadowColor = hsl(hue, sat+12, 55, 0.5);
      ctx.shadowBlur = 10 + 14*Math.sin(progress*Math.PI + t);
      ctx.fill();
    }

    ctx.restore();
  }
}

const arms = [];
for (let i=0; i<ARMS; i++) {
  arms.push(new ArmWave(
    (2*Math.PI/ARMS)*i, 
    i * 0.9
  ))
}

function animate(t) {
  t = t*0.001;

  // Fade background with very slight persistence for trails
  ctx.globalAlpha = 0.13;
  ctx.fillStyle = '#18152b';
  ctx.fillRect(0,0,W,H);
  ctx.globalAlpha = 1;

  // Draw radiating subtle background fractal circles
  for (let i=0; i<9; i++) {
    const rf = 0.38 + i*0.065 + Math.sin(t*0.79+i)*0.02;
    const r = ARM_LENGTH*rf + BASE_RADIUS + Math.sin(t*0.37+i)*9;
    ctx.beginPath();
    ctx.arc(CX, CY, r, 0, 2*Math.PI);
    ctx.strokeStyle = hsl(200+i*15+(t*40)%360,35,17+9*Math.sin(t+i),0.14);
    ctx.lineWidth = 2 + Math.sin(t*i*0.19)*0.5;
    ctx.stroke();
  }

  // Arms
  for (const arm of arms) {
    arm.draw(t);
  }

  // Optionally, central breathing circle
  ctx.save();
  ctx.beginPath();
  let ccRad = 52 + 19*Math.sin(t*0.74);
  ctx.arc(CX, CY, ccRad, 0, 2*Math.PI);
  ctx.fillStyle = hsl(
    (t*48)%360, 
    78+15*Math.sin(t*0.6), 
    62+12*Math.cos(t*0.9)
  );
  ctx.shadowColor = 'white';
  ctx.shadowBlur = 13;
  ctx.globalAlpha = 0.78 + 0.2*Math.sin(t);
  ctx.fill();
  ctx.restore();

  requestAnimationFrame(animate);
}

// Kick off
// Initial opaque fill (no trails at beginning)
ctx.fillStyle = '#18152b';
ctx.fillRect(0,0,W,H);

animate(0);
</script>
