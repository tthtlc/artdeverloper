---
layout: fullscreen
title: "Evolving Psychedelic Canvas Dreams"
tags:
  - graphics
date: 2025-01-04
---

<canvas id="psychedelic"></canvas>

<script>
const canvas = document.getElementById('psychedelic');
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

// Psychedelic render params
const shapes = 11;
const freqX = 0.2, freqY = 0.15, freqT = 0.09;
const points = 36;
let t0 = performance.now();

function draw(now) {
  const t = (now-t0)*0.00021;
  ctx.globalCompositeOperation = "lighter";
  ctx.clearRect(0, 0, width, height);

  // Evolving, overlapping, morphing patterns
  for (let s = 0; s < shapes; ++s) {
    const hue = ((t*20 + s*360/shapes) % 360);
    const alpha = 0.14 + 0.08*Math.sin(t + s);
    ctx.beginPath();
    for (let i = 0; i <= points; ++i) {
      let angle = 2 * Math.PI * i / points;
      // Dynamic radii for twirling effect
      let r = (
        0.35 + 0.12*Math.sin(t*2 + s*1.8 + angle*freqX + Math.sin(t + s))
        + 0.19*Math.cos(t*3.5 - angle*freqY + s*freqT)
        + 0.09*Math.sin(4*angle + t+s)
        + 0.08*Math.sin(t*7 - angle*1.7)
      );
      r = r * Math.min(width, height) * 0.41;
      const x = width*0.5 + r * Math.cos(angle + s+t*0.6) + Math.sin(t*6+s*1.2)*30;
      const y = height*0.5 + r * Math.sin(angle - s*1.38 - t*0.9) + Math.cos(t*4.1+s*0.9)*30;
      if (i===0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);
    }
    ctx.closePath();
    const rgb = hsv2rgb(hue, 0.9, 0.87);
    ctx.shadowBlur = 20 + 13*Math.abs(Math.sin(t+s));
    ctx.shadowColor = `rgba(${(rgb[0]*240).toFixed()}, ${(rgb[1]*240).toFixed()}, ${(rgb[2]*240).toFixed()},0.76)`;
    ctx.fillStyle = `rgba(${(rgb[0]*255).toFixed()}, ${(rgb[1]*255).toFixed()}, ${(rgb[2]*255).toFixed()}, ${alpha})`;
    ctx.fill();
  }

  // Subtle overlay for a dreamy vibe
  ctx.globalCompositeOperation = "lighter";
  ctx.fillStyle = `rgba(30,6,50,0.05)`;
  ctx.fillRect(0,0,width,height);

  requestAnimationFrame(draw);
}
draw(t0);
</script>
