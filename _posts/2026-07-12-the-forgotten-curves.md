---
layout: post
title: "The Forgotten Curves: From Ancient Greek Angle-Trisectors to Theodorus' Spiral"
tags:
  - quadratrix
  - kappa-curve
  - tschirnhausen
  - theodorus
  - semicubical
  - interactive
  - animation
date: 2026-07-12
math: true
---

Not every famous curve is a household name. Some were invented to solve a specific ancient problem and then forgotten. Others bear the names of mathematicians who barely studied them. Still others are so simple you wonder why they weren't discovered earlier. Here are eight of the most intriguing lesser-known curves — each with a story worth telling.

<div class="curve-selector" style="margin: 1em 0;">
  <button onclick="selectCurve(0)" style="margin:2px;padding:6px 12px;cursor:pointer;">Kappa Curve</button>
  <button onclick="selectCurve(1)" style="margin:2px;padding:6px 12px;cursor:pointer;">Tschirnhausen Cubic</button>
  <button onclick="selectCurve(2)" style="margin:2px;padding:6px 12px;cursor:pointer;">Semicubical Parabola</button>
  <button onclick="selectCurve(3)" style="margin:2px;padding:6px 12px;cursor:pointer;">Quadratrix of Hippias</button>
  <button onclick="selectCurve(4)" style="margin:2px;padding:6px 12px;cursor:pointer;">Theodorus Spiral</button>
  <button onclick="selectCurve(5)" style="margin:2px;padding:6px 12px;cursor:pointer;">Evolute of Ellipse</button>
  <button onclick="selectCurve(6)" style="margin:2px;padding:6px 12px;cursor:pointer;">Versiera</button>
  <button onclick="selectCurve(7)" style="margin:2px;padding:6px 12px;cursor:pointer;background:#ff6b6b;color:#fff;">Tour All</button>
</div>

<style>
@media print {
  body * { visibility: hidden; }
  #canvas-wrap6, #canvas-wrap6 * { visibility: visible; }
  #canvas-wrap6 {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    background: #fff;
  }
  #canvas-wrap6 canvas { max-width: 100vw; max-height: 100vh; width: auto; height: auto; }
}
#canvas-wrap6 {
  position: relative;
  display: inline-block;
  cursor: pointer;
  border: 1px solid #333;
  border-radius: 8px;
  overflow: hidden;
  line-height: 0;
}
#canvas-wrap6.fullscreen {
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
#canvas-wrap6 .close-btn { display: none; }
#canvas-wrap6.fullscreen .close-btn {
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

<div id="canvas-wrap6">
  <span class="close-btn">&times;</span>
  <canvas id="canvas6"></canvas>
</div>
<br/>
<label for="paramSlider6">Parameter: <span id="paramVal6">0.50</span></label>
<input type="range" id="paramSlider6" min="5" max="95" value="50" style="width:100%;" />

<script>
(function(){
const wrap = document.getElementById('canvas-wrap6');
const canvas = document.getElementById('canvas6');
const ctx = canvas.getContext('2d');
const slider = document.getElementById('paramSlider6');
const paramVal = document.getElementById('paramVal6');

let W, H, cx, cy, scale, time = 0, animId;
let curveIdx = 0;
let tourMode = false;
let tourTimer = 0;

function size() {
  const dpr = window.devicePixelRatio || 1;
  W = Math.min(700, window.innerWidth - 32);
  H = Math.min(500, W * 0.75);
  canvas.width = W * dpr;
  canvas.height = H * dpr;
  canvas.style.width = W + 'px';
  canvas.style.height = H + 'px';
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  cx = W / 2;
  cy = H / 2;
  scale = Math.min(W, H) * 0.35;
}

const curves = [
  {
    name: 'Kappa Curve', math: 'Joseph Gergonne, 1813',
    eq: function(t, k) {
      const a = 0.4 + k * 1.2;
      const st = Math.sin(t);
      const ct = Math.cos(t);
      if (Math.abs(st) < 0.005) return { x: 0, y: 0 };
      return { x: a * ct / st, y: a * ct };
    },
    range: [0.08, Math.PI - 0.08],
    desc: 'Resembles the Greek letter κ (kappa). Two branches, asymptotic to the y-axis.',
    color: '#a29bfe'
  },
  {
    name: 'Tschirnhausen Cubic', math: 'Ehrenfried von Tschirnhaus, 1682',
    eq: function(t, k) {
      const a = 0.3 + k * 1.0;
      return {
        x: a * (3 - t * t),
        y: a * t * (3 - t * t)
      };
    },
    range: [-2.2, 2.2],
    desc: 'A cubic curve with a single cusp. Tschirnhaus is better known for his work on equations.',
    color: '#4ecdc4'
  },
  {
    name: 'Semicubical Parabola', math: '17th-century geometers',
    eq: function(t, k) {
      const a = 0.3 + k * 1.2;
      return { x: t * t, y: a * t * t * t };
    },
    range: [-1.8, 1.8],
    desc: 'y² = x³ — the simplest curve with a cusp. An evolutionary dead-end in curve taxonomy.',
    color: '#f9ca24'
  },
  {
    name: 'Quadratrix of Hippias', math: 'Hippias of Elis, c. 420 BCE',
    eq: function(t, k) {
      const a = 0.5 + k * 1.5;
      const tt = t * 0.99; // avoid exact π/2
      if (Math.abs(Math.cos(tt)) < 0.001) return { x: a * 2 / Math.PI, y: tt };
      return { x: a * tt / Math.tan(tt) / Math.PI * 2, y: a * tt / Math.PI * 2 - a };
    },
    range: [-Math.PI * 0.48, Math.PI * 0.48],
    desc: 'The first curve ever invented for a specific purpose: trisecting angles and squaring the circle.',
    color: '#fd79a8'
  },
  {
    name: 'Spiral of Theodorus', math: 'Theodorus of Cyrene, c. 400 BCE',
    eq: function(t, k) {
      // Build discrete points: right triangles with hypotenuse √n
      const pts = [];
      let x = 0, y = 0, angle = 0;
      const nMax = Math.floor(3 + k * 25); // up to ~26 triangles
      for (let n = 1; n <= nMax; n++) {
        const r = Math.sqrt(n);
        const prevR = Math.sqrt(n - 1 || 1);
        angle += Math.atan2(1, Math.sqrt(n - 1 || 1));
        x += Math.cos(angle);
        y += Math.sin(angle);
        pts.push({ x: x, y: y });
      }
      return { pts: pts };
    },
    desc: 'Built from right triangles, each with hypotenuse √n. A discrete spiral of roots.',
    color: '#00b894'
  },
  {
    name: 'Evolute of Ellipse', math: 'Christiaan Huygens, 1673',
    eq: function(t, k) {
      const a = 1.6, b = 0.6 + k * 1.0;
      const ct = Math.cos(t), st = Math.sin(t);
      const denom = a * a * st * st + b * b * ct * ct;
      if (Math.abs(denom) < 0.001) return { x: 0, y: 0 };
      const x0 = a * ct, y0 = b * st;
      const curv = Math.pow(a * a * st * st + b * b * ct * ct, 1.5) / (a * b);
      const nx = -b * ct / Math.sqrt(b * b * ct * ct + a * a * st * st);
      const ny = -a * st / Math.sqrt(b * b * ct * ct + a * a * st * st);
      return { x: x0 + curv * nx, y: y0 + curv * ny };
    },
    desc: 'The locus of curvature centres of an ellipse. Astroid-like, but distinct.',
    color: '#74b9ff'
  },
  {
    name: 'Versiera (Simplified Witch)', math: 'Maria Agnesi, 1748',
    eq: function(t, k) {
      const a = 0.3 + k * 1.5;
      return { x: t, y: a / (1 + t * t) };
    },
    range: [-3, 3],
    desc: 'A simplified parametric form of the Witch of Agnesi. Bell-shaped and elegant.',
    color: '#e17055'
  }
];

function selectCurve(idx) {
  if (idx === 7) { tourMode = !tourMode; if (tourMode) tourTimer = 0; return; }
  tourMode = false;
  curveIdx = idx;
}
	window.selectCurve = selectCurve;
function drawCurve(cx0, cy0, sc, curve, tRange, alpha, k) {
  if (curve.eq.length === 0) return;
  const N = 600;

  // Special handling for Theodorus (discrete)
  const result = curve.eq(0, k);
  if (result && result.pts) {
    ctx.strokeStyle = curve.color;
    ctx.globalAlpha = alpha;
    ctx.lineWidth = 2;
    ctx.shadowColor = curve.color;
    ctx.shadowBlur = 3;
    ctx.beginPath();
    const pts = result.pts;
    // Scale to fit
    const maxR = Math.sqrt(pts.length + 1);
    const s = sc * 0.7;
    ctx.moveTo(cx0, cy0);
    for (let i = 0; i < pts.length; i++) {
      const sx = cx0 + pts[i].x * s;
      const sy = cy0 - pts[i].y * s;
      ctx.lineTo(sx, sy);
    }
    ctx.stroke();
    // Draw dots at vertices
    ctx.fillStyle = curve.color;
    for (let i = 0; i < pts.length; i++) {
      const sx = cx0 + pts[i].x * s;
      const sy = cy0 - pts[i].y * s;
      ctx.beginPath();
      ctx.arc(sx, sy, 2.5, 0, Math.PI * 2);
      ctx.fill();
    }
    ctx.shadowBlur = 0;
    ctx.globalAlpha = 1;
    return;
  }

  ctx.beginPath();
  let first = true;
  for (let i = 0; i <= N; i++) {
    const t = tRange[0] + (i / N) * (tRange[1] - tRange[0]);
    try {
      const pt = curve.eq(t, k);
      if (!pt || !isFinite(pt.x) || !isFinite(pt.y)) { first = true; continue; }
      if (pt.pts) continue;
      const sx = cx0 + pt.x * sc;
      const sy = cy0 - pt.y * sc;
      if (Math.abs(sx - cx0) > W || Math.abs(sy - cy0) > H) { first = true; continue; }
      if (first) { ctx.moveTo(sx, sy); first = false; }
      else { ctx.lineTo(sx, sy); }
    } catch(e) { first = true; }
  }
  ctx.strokeStyle = curve.color;
  ctx.globalAlpha = alpha;
  ctx.lineWidth = 2.5;
  ctx.shadowColor = curve.color;
  ctx.shadowBlur = 4;
  ctx.stroke();
  ctx.shadowBlur = 0;
  ctx.globalAlpha = 1;
}

function draw() {
  ctx.clearRect(0, 0, W, H);
  const k = slider.value / 100;

  // Grid
  ctx.strokeStyle = 'rgba(255,255,255,0.06)';
  ctx.lineWidth = 0.5;
  for (let i = -6; i <= 6; i++) {
    ctx.beginPath(); ctx.moveTo(cx + i * scale * 0.3, 0); ctx.lineTo(cx + i * scale * 0.3, H); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(0, cy + i * scale * 0.3); ctx.lineTo(W, cy + i * scale * 0.3); ctx.stroke();
  }
  ctx.strokeStyle = 'rgba(255,255,255,0.12)';
  ctx.lineWidth = 1;
  ctx.beginPath(); ctx.moveTo(0, cy); ctx.lineTo(W, cy); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(cx, 0); ctx.lineTo(cx, H); ctx.stroke();

  if (tourMode) {
    tourTimer++;
    if (tourTimer > 300) { tourTimer = 0; curveIdx = (curveIdx + 1) % 7; }
  }

  const curve = curves[curveIdx];
  const tRange = curve.range || [0, Math.PI * 2];
  drawCurve(cx, cy, scale, curve, tRange, 1, k);

  // Tracer (skip for Theodorus)
  const result = curve.eq(0, k);
  if (!result || !result.pts) {
    const traceT = tRange[0] + ((time * 0.025) % 1) * (tRange[1] - tRange[0]);
    try {
      const dp = curve.eq(traceT, k);
      if (dp && isFinite(dp.x) && isFinite(dp.y) && !dp.pts) {
        const dx = cx + dp.x * scale;
        const dy = cy - dp.y * scale;
        ctx.beginPath();
        ctx.arc(dx, dy, 5, 0, Math.PI * 2);
        ctx.fillStyle = '#fff';
        ctx.fill();
        ctx.strokeStyle = curve.color;
        ctx.lineWidth = 2;
        ctx.stroke();
      }
    } catch(e) {}
  }

  ctx.fillStyle = '#fff';
  ctx.font = 'bold 15px "Segoe UI", sans-serif';
  ctx.textAlign = 'center';
  const label = tourMode ? 'Tour: ' + curve.name : curve.name;
  ctx.fillText(label, cx, 18);
  ctx.font = '11px "Segoe UI", sans-serif';
  ctx.fillStyle = '#aaa';
  ctx.fillText(curve.math, cx, 34);
  ctx.fillText(curve.desc, cx, H - 10);

  paramVal.textContent = k.toFixed(2);
  time++;
  animId = requestAnimationFrame(draw);
}

wrap.addEventListener('click', function(e) {
  if (e.target === wrap || e.target === canvas) {
    if (document.fullscreenElement) { document.exitFullscreen(); }
    else { wrap.requestFullscreen(); }
  }
});
wrap.querySelector('.close-btn').addEventListener('click', function(e) {
  e.stopPropagation();
  if (document.fullscreenElement) document.exitFullscreen();
});
document.addEventListener('fullscreenchange', function() {
  if (document.fullscreenElement === wrap) { wrap.classList.add('fullscreen'); }
  else { wrap.classList.remove('fullscreen'); }
  setTimeout(size, 100);
});
window.addEventListener('resize', size);
size();
draw();
})();
</script>

---

## The Quadratrix: The First Curve With a Mission

Around 420 BCE, Hippias of Elis set out to solve one of the three classical problems of Greek geometry: **trisecting an angle**. He invented what is probably the first curve in history defined not by a geometric construction but by a *kinematic* description — the simultaneous motion of two lines, one rotating and one translating.

The quadratrix is defined implicitly: a horizontal line moves downward at constant speed while a radius rotates clockwise at constant speed. Their intersection traces the curve. Once you have the quadratrix, you can trisect any angle — and even square the circle (hence the name).

It caused a philosophical scandal. Greek geometers insisted curves should be constructible with compass and straightedge; the quadratrix required a *mechanical* motion. It was a curve ahead of its time.

## The Spiral of Theodorus: Building √n, One Triangle at a Time

Theodorus of Cyrene (Socrates' math teacher) constructed this discrete spiral around 400 BCE. Start with a right triangle of legs 1 and 1 (hypotenuse √2). Build another right triangle using √2 as one leg and 1 as the other (hypotenuse √3). Continue: each step adds a triangle whose hypotenuse is √n.

The resulting spiral of points winds outward, each vertex at distance √n from the origin. Theodorus proved the irrationality of √3, √5, ..., √17 using this construction — and then, mysteriously, stopped at √17. Why? Mathematicians still debate this. The spiral itself approximates an Archimedean spiral for large n, but its discrete nature gives it a unique jagged beauty.

## The Semicubical Parabola: A Cusp and Nothing More

\\(y^2 = x^3\\) is one of the simplest curves with a **cusp** — a sharp point where the curve reverses direction. It has no loops, no asymptotes, no petals. Its very simplicity made it important: it was one of the first examples where mathematicians recognised that a curve could be smooth everywhere except at isolated singularities.

## The Kappa Curve: Gergonne's Greek Letter

Joseph Gergonne named this curve for its resemblance to κ (kappa). Given by \\(y^2(a^2 - x^2) = a^2 x^2\\), it has two branches that extend vertically, approaching the y-axis asymptotically but never touching it. Gergonne was the founding editor of the first purely mathematical journal, *Annales de Mathématiques*, in 1810. The kappa curve was one of dozens he catalogued.

## Tschirnhausen's Cubic: A Polynomial With Style

Ehrenfried von Tschirnhaus is mostly remembered for the Tschirnhaus transformation — a method for eliminating intermediate terms from polynomial equations. His cubic curve, \\(27ay^2 = (a - x)(x + 3a)^2\\), is a lesser legacy but a beautiful one, with a single cusp and a graceful loop.

## The Evolute of an Ellipse

The evolute is the envelope of normals — or equivalently, the locus of all centres of curvature. The evolute of an ellipse is a stretched astroid with four cusps, each corresponding to a point of extreme curvature on the ellipse. Huygens used evolutes in his design of pendulum clocks: by making the pendulum bob follow a cycloidal evolute path, he made the period independent of amplitude — the first isochronous pendulum.

---

**Try it:** Click buttons to switch between curves. Move the slider to modify each curve's defining parameter. Hit **Tour All** for a hands-free journey through every forgotten curve.

<style>
.curve-selector button { transition: all 0.2s; }
.curve-selector button:hover { transform: scale(1.05); }
</style>
