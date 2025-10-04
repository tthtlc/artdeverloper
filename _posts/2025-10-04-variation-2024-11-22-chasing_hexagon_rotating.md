```markdown
---
layout: fullscreen
title: "Psychedelic Wave Vortex with Colorful Particles"
tags:
  - graphics
---

<canvas id="vortexCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('vortexCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width;
const H = canvas.height;
const center = { x: W / 2, y: H / 2 };

let t = 0;

// Palette: Neon rainbow plus black for depth
const palette = [
  "#ff00cc", "#ffea00", "#40ff00", "#00ffd6",
  "#0076ff", "#7f00ff", "#ff007f", "#ffffff", "#000000"
];

// Particle system settings
const NUM_PARTICLES = 100;
const particles = [];

// Initialize spiral particles
function initParticles() {
  for (let i = 0; i < NUM_PARTICLES; i++) {
    const angle = (2 * Math.PI / NUM_PARTICLES) * i;
    const dist = 120 + Math.random() * 100;
    const speed = 0.004 + Math.random() * 0.004;
    const phase = Math.random() * Math.PI * 2;
    const color = palette[i % palette.length];
    particles.push({
      baseAngle: angle,
      dist,
      speed,
      phase,
      color,
      size: 2 + Math.random() * 4
    });
  }
}
initParticles();

// Draw a sinuous vortex with undulating lines 
function drawVortexWave(time) {
  const layers = 8;
  for (let l = 0; l < layers; l++) {
    ctx.save();
    ctx.translate(center.x, center.y);
    const baseRadius = (l + 1) * 30 + 10 * Math.sin(time/410 + l);
    const twist = Math.sin(time/380 + l*0.8) * Math.PI;

    ctx.rotate(twist);

    ctx.beginPath();
    for (let a = 0; a <= Math.PI * 2 + 0.12; a += 0.11) {
      const mod = 1 + 0.13*Math.sin(4*a + time/90 + l*0.9);
      const dev = 16 * Math.sin(6*a + time/140 + l);
      const x = Math.cos(a) * (baseRadius * mod + dev);
      const y = Math.sin(a) * (baseRadius * mod + dev);
      if (a === 0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);
    }
    // Color: Shift through palette dynamically
    const gradient = ctx.createLinearGradient(-baseRadius, -baseRadius, baseRadius, baseRadius);
    palette.forEach((col, i) => {
      gradient.addColorStop(i / (palette.length-1), col);
    });
    ctx.strokeStyle = gradient;
    ctx.globalAlpha = 0.65;
    ctx.lineWidth = 3 + 2*Math.sin(time/320 + l*0.6);
    ctx.shadowColor = "#000000";
    ctx.shadowBlur = 16 - l*1.5;
    ctx.stroke();
    ctx.restore();
  }
}

// Animate spiral particles along the wave
function drawParticles(time) {
  for (let i = 0; i < particles.length; i++) {
    const p = particles[i];
    // Animate along spiral
    const angle = p.baseAngle + (time * p.speed) + Math.sin(time/120 + i);
    const rmod = 1.18 + 0.16*Math.sin(5*angle + time/700 + i);
    const r = p.dist * rmod + 40*Math.sin(time/330 + i);
    const x = center.x + r * Math.cos(angle);
    const y = center.y + r * Math.sin(angle);
    // Pulse for psychedelic effect
    const pulse = 0.7 + 0.25*Math.sin(time/45 + i);

    // Glow
    ctx.save();
    ctx.beginPath();
    ctx.arc(x, y, p.size * pulse, 0, Math.PI * 2);
    ctx.closePath();
    ctx.globalAlpha = 0.7 + 0.3*Math.sin(time/160 + i);
    ctx.fillStyle = p.color;
    ctx.shadowColor = p.color;
    ctx.shadowBlur = 16 + 16*Math.sin(time/140 + i);
    ctx.fill();
    ctx.restore();
  }
}

// Black transparent overlay to fade traces (motion blur effect)
function fadeCanvas() {
  ctx.save();
  ctx.globalAlpha = 0.11;
  ctx.fillStyle = "#000000";
  ctx.fillRect(0, 0, W, H);
  ctx.restore();
}

// Animation loop
function animate() {
  fadeCanvas(); // creates glowing trails
  drawVortexWave(t);
  drawParticles(t);

  t += 1;
  requestAnimationFrame(animate);
}

// Start with a black background
ctx.fillStyle = "#000000";
ctx.fillRect(0, 0, W, H);

animate();
</script>
```