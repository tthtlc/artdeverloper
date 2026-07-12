---
layout: post
title: "Reversing Hexagon Spirals"
tags:
  - hexagon
  - spiral
  - outward-spiral
  - reversing-spiral
  - interactive
  - animation
date: 2026-07-12
---

Five outward spirals radiate from a single centre, each tracing a hexagon that expands step by step. The curvature of each spiral slowly oscillates — clockwise tightens, then unwinds through straight-radial, then curls counterclockwise, then unwinds again and reverses — so the spirals breathe between handedness in a continuous, hypnotic cycle.

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

let W, H, cx, cy, time = 0, animId;

function size() {
  const full = wrap.classList.contains('fullscreen');
  const dpr = window.devicePixelRatio || 1;
  W = full ? window.innerWidth : 600;
  H = full ? window.innerHeight : 600;
  canvas.width  = W * dpr;
  canvas.height = H * dpr;
  canvas.style.width  = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  cx = W / 2; cy = H / 2;
}

/* ---- tiny starting hexagon ---- */
function hexPoints(cx, cy, r, rot) {
  const pts = [];
  for (let i = 0; i < 6; i++) {
    const a = (Math.PI / 3) * i + rot;
    pts.push({ x: cx + r * Math.cos(a), y: cy + r * Math.sin(a) });
  }
  return pts;
}

/* ---- outward spiral from one hexagon ---- */
function drawOutwardSpiral(initialPts, iters, t, hueBase, phaseOffset, startRot) {
  let pts = initialPts;

  // period of one full curvature cycle (seconds)
  const period = 14;
  // maximum rotation per step in radians  (~3.7°)
  const maxAngle = 0.065;
  // expansion per step — small so the spiral is dense
  const expand = 1.026;

  for (let k = 0; k < iters; k++) {
    /* ----- draw the six edges of the current hexagon ----- */
    for (let i = 0; i < pts.length; i++) {
      const a = pts[i];
      const b = pts[(i + 1) % pts.length];

      // early skip when the edge is far outside the visible area
      const da = Math.hypot(a.x - cx, a.y - cy);
      const db = Math.hypot(b.x - cx, b.y - cy);
      if (Math.min(da, db) > Math.max(W, H) * 0.75) continue;

      ctx.beginPath();
      ctx.moveTo(a.x, a.y);
      ctx.lineTo(b.x, b.y);

      // colour: hue rotates with iteration, edge index and time
      const hue   = (hueBase + k * 4 + i * 22 + t * 18) % 360;
      const distF = Math.min(Math.max(da, db) / (Math.min(W, H) * 0.45), 1);
      const sat   = 60 + 35 * Math.sin(t * 0.7 + k * 0.10 + phaseOffset);
      const light = 40 + 28 * (1 - distF) + 12 * Math.sin(t * 0.5 + i);

      ctx.strokeStyle = `hsl(${hue}, ${sat}%, ${light}%)`;
      ctx.lineWidth   = 0.55 + 0.45 * (1 - distF);
      ctx.stroke();
    }

    /* ----- oscillating curvature -----
       rotAngle > 0  → clockwise spiral
       rotAngle ≈ 0  → straight radial expansion
       rotAngle < 0  → counter‑clockwise spiral                */
    const rotAngle = maxAngle * Math.sin(t * (2 * Math.PI) / period + phaseOffset);

    const cosA = Math.cos(rotAngle);
    const sinA = Math.sin(rotAngle);

    /* ----- build the next (larger, rotated) hexagon ----- */
    const next = [];
    for (let i = 0; i < pts.length; i++) {
      const dx = pts[i].x - cx;
      const dy = pts[i].y - cy;
      next.push({
        x: cx + (dx * cosA - dy * sinA) * expand,
        y: cy + (dx * sinA + dy * cosA) * expand
      });
    }
    pts = next;
  }
}

/* ---- animation loop ---- */
function draw(timestamp) {
  time = timestamp * 0.001;
  ctx.clearRect(0, 0, W, H);

  // deep indigo background
  ctx.fillStyle = '#07071a';
  ctx.fillRect(0, 0, W, H);

  const startRadius = Math.min(W, H) * 0.018;
  const iters       = 110;

  // soft centre glow
  const glow = ctx.createRadialGradient(cx, cy, 0, cx, cy, startRadius * 4);
  glow.addColorStop(0, 'rgba(200,210,255,0.35)');
  glow.addColorStop(1, 'rgba(200,210,255,0)');
  ctx.fillStyle = glow;
  ctx.beginPath(); ctx.arc(cx, cy, startRadius * 4, 0, Math.PI * 2); ctx.fill();

  // slow global rotation of the seed hexagon
  const globalRot = time * 0.12;

  // --- five spirals, each with a different curvature phase ---
  const spirals = 5;
  for (let s = 0; s < spirals; s++) {
    // curvature phase — evenly spread across one full cycle
    const phase   = (Math.PI * 2 / spirals) * s;
    // each spiral starts from a hexagon rotated a few degrees
    // differently so the arms separate visually
    const startRot = globalRot + (Math.PI / 3 / spirals) * s * 0.7;
    const seed     = hexPoints(cx, cy, startRadius, startRot);
    drawOutwardSpiral(seed, iters, time, s * 72, phase, startRot);
  }

  // subtle centre dot
  ctx.fillStyle = 'rgba(255,255,255,0.5)';
  ctx.beginPath(); ctx.arc(cx, cy, 2.5, 0, Math.PI * 2); ctx.fill();

  animId = requestAnimationFrame(draw);
}

/* ---- bootstrap ---- */
size();
animId = requestAnimationFrame(draw);

/* ---- fullscreen toggle (same UX as the original) ---- */
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

Instead of seven nested hexagons chasing inward, this variation uses **five outward-expanding hexagon spirals that all begin from a single centre point**.

### Outward expansion with rotation

Each spiral starts from a tiny seed hexagon at the canvas centre. At every iteration the hexagon is:

1. **scaled up** by a small constant factor (~2.6 % per step), pushing it outward,
2. **rotated** by a *curvature angle* that varies over time.

A positive curvature angle rotates each successive hexagon clockwise relative to the previous one, producing a **clockwise outward spiral**. A negative angle produces a **counter‑clockwise outward spiral**. When the angle is near zero the hexagons march outward in nearly straight radial lines — the spiral "unwinds."

### Curvature oscillation

The curvature angle follows a smooth sine wave with a period of about 14 seconds:

```
curvature(t) = maxAngle × sin(2π·t / period + phaseOffset)
```

- The angle **decreases** toward zero → the spiral loosens, becoming more radial.
- It **crosses zero** → the handedness reverses (clockwise ↔ counter‑clockwise).
- It **increases** in the opposite direction → the reverse spiral tightens.
- Then the cycle repeats in the other direction.

Because each of the five spirals has a different `phaseOffset`, at any instant some are curling clockwise, some counter‑clockwise, and some passing through the radial "straight" state.

### Colour

Each spiral has its own base hue (stepped by 72° around the colour wheel). Within a spiral, hue shifts with iteration depth, edge index, and time. Saturation pulses gently and lines further from centre are darker, giving the pattern a sense of depth.
