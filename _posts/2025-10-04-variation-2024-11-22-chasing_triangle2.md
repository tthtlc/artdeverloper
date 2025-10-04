---
layout: fullscreen
title: "Hypnotic Rainbow Double Spiral"
tags:
  - graphics
---

<canvas id="spiralCanvas" width="600" height="600" style="background: #10101a; display:block; margin:0 auto;"></canvas>
<script>

const canvas = document.getElementById('spiralCanvas');
const ctx = canvas.getContext('2d');
const w = canvas.width;
const h = canvas.height;

// Spiral parameters
const numParticles = 320;
const spiralArms = 2;
const spiralSpread = 210;
const armSeparation = Math.PI * 2 / spiralArms;

let t = 0;

// Particle store
class SpiralParticle {
  constructor(i) {
    this.baseAngle = i * (Math.PI * 12 / numParticles);
    this.arm = i % spiralArms;
    this.radius = spiralSpread * (i / numParticles);
    this.size = 2.2 + Math.sin(i * 0.23) * 1.6;
    this.colorPhase = i / numParticles;
  }
  update(time) {
    // Radius modulates to create 'breathing'
    const r = this.radius + Math.sin(time*0.97 + this.baseAngle*0.9) * 24;
    // Angular speed creates twisting motion
    const phase = this.baseAngle + time * (0.25 + 0.12 * Math.sin(this.baseAngle*2));
    this.x = w/2 + Math.cos(phase + this.arm*armSeparation) * r;
    this.y = h/2 + Math.sin(phase + this.arm*armSeparation) * r;
    this.c = `hsl(${360 * (this.colorPhase + time * 0.09)%360}, 100%, 60%)`;
    this.alpha = 0.66 + 0.33*Math.sin(time*1.4 + this.baseAngle*3.1);
  }
  draw(ctx) {
    ctx.save();
    ctx.globalAlpha = this.alpha;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size, 0, 2 * Math.PI);
    ctx.fillStyle = this.c;
    ctx.shadowColor = this.c;
    ctx.shadowBlur = 12 + 12*Math.abs(Math.sin(this.baseAngle+this.alpha));
    ctx.fill();
    ctx.restore();
  }
}

const particles = [];
for (let i = 0; i < numParticles; i++) {
  particles.push(new SpiralParticle(i));
}

// Draw central radiating waveform
function drawCoreWave(time) {
  ctx.save();
  ctx.translate(w/2, h/2);
  ctx.rotate(time*0.17);
  for (let j = 0; j < 45; j++) {
    const angle = j * (2 * Math.PI / 45);
    const rad = 36 + 24*Math.sin(time*2 + j*0.3);
    ctx.beginPath();
    ctx.moveTo(
      Math.cos(angle) * 7, 
      Math.sin(angle) * 7
    );
    ctx.lineTo(
      Math.cos(angle) * (rad + 14*Math.sin(time*5 + j)), 
      Math.sin(angle) * (rad + 14*Math.sin(time*5 + j))
    );
    ctx.strokeStyle = `hsl(${(time*60 + j*6)%360},100%,50%)`;
    ctx.lineWidth = 3.2+(2.2*Math.sin(j+time*0.4));
    ctx.shadowColor = ctx.strokeStyle;
    ctx.shadowBlur = 18;
    ctx.stroke();
  }
  ctx.restore();
}

// Psychedelic trails
function fadeCanvas() {
  // Partial alpha fade for trails
  ctx.globalAlpha = 0.13;
  ctx.fillStyle = "#10101a";
  ctx.fillRect(0, 0, w, h);
  ctx.globalAlpha = 1;
}


function animate() {
  t += 0.012;
  fadeCanvas();

  // Draw central waveform first
  drawCoreWave(t);

  // Draw double spiral particles
  for (let i = 0; i < particles.length; i++) {
    particles[i].update(t);
    particles[i].draw(ctx);
  }
  
  requestAnimationFrame(animate);
}

// Initial fill for clean fade-in
ctx.fillStyle = '#10101a';
ctx.fillRect(0, 0, w, h);

animate();

</script>
