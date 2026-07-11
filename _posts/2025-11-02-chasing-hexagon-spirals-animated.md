---
layout: post
title: "Animated Spiraling Hexagons"
tags:
  - hexagon
  - spiral
  - chasing-polygon
  - interactive
  - animation
date: 2025-11-02
---

Seven hexagons — one at the centre and six orbiting it — each filled with chasing polygons that spiral endlessly inward. The original [chasing hexagon around hexagon](/chasing_6gon_around_6gon.html) drew static spirals; this variant brings them to life with continuous animated motion.

## Interactive Demo

<style>
@media print {
  body * { visibility: hidden; }
  #canvas-wrap, #canvas-wrap * { visibility: visible; }
  #canvas-wrap {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    background: #fff;
  }
  #canvas-wrap canvas { max-width: 100vw; max-height: 100vh; width: auto; height: auto; }
}

#canvas-wrap {
  position: relative;
  display: inline-block;
  cursor: pointer;
  border: 1px solid #333;
  border-radius: 8px;
  overflow: hidden;
  line-height: 0;
}

#canvas-wrap.fullscreen {
  position: fixed;
  top: 0; left: 0; width: 100vw; height: 100vh;
  background: #000;
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  border-radius: 0;
}

#canvas-wrap .close-btn {
  display: none;
}
#canvas-wrap.fullscreen .close-btn {
  display: block;
  position: absolute;
  top: 12px; right: 16px;
  color: #fff;
  font-size: 28px;
  line-height: 1;
  cursor: pointer;
  z-index: 1;
  user-select: none;
}
</style>

<div id="canvas-wrap">
  <span class="close-btn">&times;</span>
  <canvas id="canvas"></canvas>
</div>

<script>
const wrap = document.getElementById('canvas-wrap');
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

let W, H, cx, cy, hexRadius, time = 0, animId;

function size() {
  const full = wrap.classList.contains('fullscreen');
  const dpr = window.devicePixelRatio || 1;
  W = full ? window.innerWidth : 600;
  H = full ? window.innerHeight : 600;
  canvas.width = W * dpr;
  canvas.height = H * dpr;
  canvas.style.width = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  cx = W / 2; cy = H / 2;
  hexRadius = Math.min(W, H) * 0.11;
}

function hexPoints(cx, cy, r, rot) {
  const pts = [];
  for (let i = 0; i < 6; i++) {
    const a = (Math.PI / 3) * i + rot;
    pts.push({ x: cx + r * Math.cos(a), y: cy + r * Math.sin(a) });
  }
  return pts;
}

function midpoint(a, b) {
  return { x: (a.x + b.x) / 2, y: (a.y + b.y) / 2 };
}

// Draw chasing polygons inside a hexagon — each iteration takes
// a step `frac` along every edge, forming a smaller inner polygon.
// Over many iterations the edges trace out a spiral.
function drawChasing(initial, iters, frac, t, hueBase) {
  let pts = initial;
  for (let k = 0; k < iters; k++) {
    for (let i = 0; i < pts.length; i++) {
      const a = pts[i];
      const b = pts[(i + 1) % pts.length];
      ctx.beginPath();
      ctx.moveTo(a.x, a.y);
      ctx.lineTo(b.x, b.y);
      const hue = (hueBase + k * 6 + i * 30 + t * 40) % 360;
      const sat = 70 + 30 * Math.sin(t * 1.3 + k * 0.2);
      ctx.strokeStyle = `hsl(${hue}, ${sat}%, ${55 + 10 * Math.sin(t * 0.7 + i)}%)`;
      ctx.lineWidth = 0.8;
      ctx.stroke();
    }
    // Next polygon: points `frac` along each edge
    const next = [];
    for (let i = 0; i < pts.length; i++) {
      const a = pts[i];
      const b = pts[(i + 1) % pts.length];
      next.push({
        x: a.x + (b.x - a.x) * frac,
        y: a.y + (b.y - a.y) * frac
      });
    }
    pts = next;
  }
}

function draw(timestamp) {
  time = timestamp * 0.001;
  ctx.clearRect(0, 0, W, H);

  const globalRot = time * 0.15;
  const centre = hexPoints(cx, cy, hexRadius, globalRot);

  // Oscillating fraction — smaller = tighter spiral, breathes with time
  const frac = 0.03 + 0.09 * (0.5 + 0.5 * Math.sin(time * 0.6));
  const iters = 70;

  // Central hexagon
  drawChasing(centre, iters, frac, time, 0);

  // Six surrounding hexagons — one centred on each edge midpoint,
  // rotated −60° so they nest cleanly against the central one.
  for (let i = 0; i < centre.length; i++) {
    const p1 = centre[i];
    const p2 = centre[(i + 1) % centre.length];
    const mid = midpoint(p1, p2);
    const vx = mid.x - cx;
    const vy = mid.y - cy;
    const nc = { x: cx + 2 * vx, y: cy + 2 * vy };
    const outer = hexPoints(nc.x, nc.y, hexRadius, globalRot - Math.PI / 3);
    drawChasing(outer, iters, frac, time, i * 60);
  }

  animId = requestAnimationFrame(draw);
}

size();
animId = requestAnimationFrame(draw);

// ---- fullscreen toggle ----
let lastTap = 0;
wrap.addEventListener('pointerdown', function(e) {
  const now = Date.now();
  if (now - lastTap < 400) {
    e.preventDefault();
    wrap.classList.toggle('fullscreen');
    size();
  }
  lastTap = now;
});

wrap.addEventListener('dblclick', function(e) {
  wrap.classList.toggle('fullscreen');
  size();
});

document.addEventListener('keydown', function(e) {
  if (e.key === 'Escape' && wrap.classList.contains('fullscreen')) {
    wrap.classList.remove('fullscreen');
    size();
  }
});

wrap.querySelector('.close-btn').addEventListener('click', function(e) {
  e.stopPropagation();
  wrap.classList.remove('fullscreen');
  size();
});
</script>

## How It Works

The seven-hexagon arrangement comes from the original static demo: a central hexagon with six copies placed at the midpoints of its edges, each rotated −60° and translated outward by twice the vector from centre to midpoint.

Each hexagon is filled with **chasing polygons** — at every iteration, a new polygon is formed by taking a point partway along each edge of the previous one. With a small step fraction (~0.03–0.12) and many iterations, the edges trace out a dense inward spiral.

The animation comes from two sources:

- **Oscillating fraction** — the step size breathes with a sine wave, making the spiral contract and expand rhythmically.
- **Colour cycling** — each edge is drawn in HSL space with hue rotating over time, iteration depth, and edge index. Saturation and lightness also shift gently.

A slow global rotation ties the seven hexagons together into one unified, hypnotic motion.
