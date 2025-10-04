```markdown
---
layout: fullscreen
title: "Hypnotic Lissajous Nebula"
tags:
  - graphics
---

<canvas id="lissajousCanvas" width="700" height="700" style="background: #16161a; display: block; margin: 0 auto; touch-action: none;"></canvas>

<script>
const canvas = document.getElementById('lissajousCanvas');
const ctx = canvas.getContext('2d');
const w = canvas.width, h = canvas.height;
const cx = w / 2, cy = h / 2;

// Parameters for the evolving Lissajous figures
const shapeCount = 9; // number of overlapping shapes
const baseRadius = 220;
const pointCount = 160; // points per path
const colorStops = [
  [0, '#60e8fc'],   // cyan
  [0.32, '#ffd972'], // yellow
  [0.52, '#ff19a3'], // pink
  [0.84, '#7cfc60'],  // lime
  [1, '#60e8fc'],
];

// Utility: interpolate between two hex colors
function lerpColor(a, b, t) {
  const ah = parseInt(a.replace('#',''),16), bh = parseInt(b.replace('#',''),16);
  const ar = ah>>16, ag = (ah>>8)&0xff, ab = ah&0xff;
  const br = bh>>16, bg = (bh>>8)&0xff, bb = bh&0xff;
  return `rgb(${(ar+(br-ar)*t)|0}, ${(ag+(bg-ag)*t)|0}, ${(ab+(bb-ab)*t)|0})`;
}

// Create a looping rainbow from gradient stops
function getColor(t) {
  t = (t % 1 + 1) % 1;
  for (let i=1; i < colorStops.length; i++) {
    let [a, ca] = colorStops[i-1], [b, cb] = colorStops[i];
    if (t <= b) return lerpColor(ca, cb, (t - a)/(b-a));
  }
  return colorStops[colorStops.length-1][1];
}

// Returns a Lissajous curve as an array of points
function lissajousPath(a, b, delta, t, R, phase, offset) {
  const pts = [];
  for (let i=0; i<pointCount; i++) {
    let th = 2 * Math.PI * i / pointCount;
    let x = R * Math.sin(a * th + phase + offset)
          + R/3 * Math.cos(a * .5 * th + 2 * offset)
          + R*.08 * Math.cos(8*th + t) ;
    let y = R * Math.sin(b * th + delta + 2*phase)
          + R/3 * Math.cos(b * .5 * th - offset)
          + R*.08 * Math.sin(8*th - t);
    pts.push([cx + x, cy + y]);
  }
  return pts;
}

// Draw a closed glowing path
function drawGlowPath(pts, col, blur=16, alpha=0.1) {
  ctx.save();
  ctx.beginPath();
  ctx.moveTo(pts[0][0], pts[0][1]);
  for(let i=1; i<pts.length; i++)
    ctx.lineTo(pts[i][0], pts[i][1]);
  ctx.closePath();
  ctx.shadowColor = col;
  ctx.shadowBlur = blur;
  ctx.globalAlpha = alpha;
  ctx.strokeStyle = col;
  ctx.lineWidth = 4;
  ctx.stroke();
  ctx.globalAlpha = 1;
  ctx.restore();
}

// Draw halo dots along a path
function drawHaloDots(pts, glowColor, t) {
  for (let i = 0; i < pts.length; i += 8) {
    let [x, y] = pts[i];
    let pulse = 0.7 + 0.3*Math.sin(t + i*.09);
    ctx.save();
    ctx.beginPath();
    ctx.arc(x, y, 5*pulse, 0, 2*Math.PI);
    ctx.fillStyle = glowColor;
    ctx.globalAlpha = .1 * pulse;
    ctx.shadowColor = glowColor;
    ctx.shadowBlur = 18 + 24*pulse;
    ctx.fill();
    ctx.restore();
  }
}

let frame = 0;

function animate() {
  frame++;
  const t = frame * 0.012;
  ctx.clearRect(0,0,w,h);

  // Dynamic parameters
  for (let s = 0; s < shapeCount; s++) {
    // Animate Lissajous parameters
    let freqA = 2 + Math.sin(t*.9 + s*.31)*2 + (s%2);
    let freqB = 3 + Math.cos(t*.62 + s*.521)*2 + (s%3? 0.5 : 0);
    let phase = (t*0.3 + s*0.22) % (2*Math.PI);
    let delta = t*0.09 + s*0.17;
    let radius = baseRadius - 30*Math.sin(t*0.47 + s*.51);

    // Color for the curve
    let baseColor = getColor((s/shapeCount + t*0.1 + 0.4*Math.sin(s)) % 1);

    // Get points and draw glowing curve
    let path = lissajousPath(freqA, freqB, delta, t, radius, phase, s*1.4);
    drawGlowPath(path, baseColor, 20, 0.16);

    // Draw intricate inner line (rotational symmetry)
    ctx.save();
    ctx.beginPath();
    for (let k=0; k<5; k++) {
      let idx = ((k*pointCount/5 + (frame + s*17)%pointCount)|0) % pointCount;
      ctx.moveTo(cx, cy);
      ctx.lineTo(path[idx][0], path[idx][1]);
    }
    ctx.lineWidth = 1.2;
    ctx.strokeStyle = baseColor;
    ctx.globalAlpha = 0.13;
    ctx.shadowColor = baseColor;
    ctx.shadowBlur = 18;
    ctx.stroke();
    ctx.restore();

    // Draw animated halo dots along the curve
    drawHaloDots(path, baseColor, t*1.1 + s*0.33);
  }

  // Center nebulous energy
  ctx.save();
  ctx.beginPath();
  ctx.arc(cx, cy, 60 + 16*Math.sin(frame*0.045), 0, 2*Math.PI);
  ctx.globalAlpha = .07;
  ctx.fillStyle = getColor((.5 + t*.21)%1);
  ctx.shadowColor = getColor((.2 + t*.38)%1);
  ctx.shadowBlur = 70;
  ctx.fill();
  ctx.restore();

  requestAnimationFrame(animate);
}

animate();

// Optional: Mouse motion distorts the nebula
let mouseX = 0, mouseY = 0;
canvas.addEventListener('pointermove', e => {
  const rect = canvas.getBoundingClientRect();
  mouseX = (e.clientX - rect.left - cx) / cx;
  mouseY = (e.clientY - rect.top - cy) / cy;
  // slightly warp center on movement
  ctx.setTransform(1,0,0,1,0,0);
  ctx.translate(cx + mouseX * 16, cy + mouseY * 16);
  ctx.translate(-cx, -cy);
});
canvas.addEventListener('pointerleave', e=> ctx.setTransform(1,0,0,1,0,0));
</script>
```
