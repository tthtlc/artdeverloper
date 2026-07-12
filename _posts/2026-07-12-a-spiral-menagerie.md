---
layout: post
title: "A Spiral Menagerie: Ten Ways to Wind Your Way to Infinity"
tags:
  - spiral
  - archimedean
  - logarithmic
  - fibonacci
  - clothoid
  - interactive
  - animation
date: 2026-07-12
math: true
---

From the shell of a nautilus to the sweep of a hurricane, spirals are nature's favourite curve. But not all spirals are created equal: some grow linearly, some exponentially, some tighten as they go. Each has its own equation, its own discoverer, and its own role in science and engineering. Here are ten spirals, side by side, winding their way to infinity.

<div class="curve-selector" style="margin: 1em 0;">
  <button onclick="selectCurve(0)" style="margin:2px;padding:6px 12px;cursor:pointer;">Archimedean</button>
  <button onclick="selectCurve(1)" style="margin:2px;padding:6px 12px;cursor:pointer;">Logarithmic</button>
  <button onclick="selectCurve(2)" style="margin:2px;padding:6px 12px;cursor:pointer;">Hyperbolic</button>
  <button onclick="selectCurve(3)" style="margin:2px;padding:6px 12px;cursor:pointer;">Fermat</button>
  <button onclick="selectCurve(4)" style="margin:2px;padding:6px 12px;cursor:pointer;">Lituus</button>
  <button onclick="selectCurve(5)" style="margin:2px;padding:6px 12px;cursor:pointer;">Cornu (Clothoid)</button>
  <button onclick="selectCurve(6)" style="margin:2px;padding:6px 12px;cursor:pointer;">Fibonacci</button>
  <button onclick="selectCurve(7)" style="margin:2px;padding:6px 12px;cursor:pointer;">Poinsot</button>
  <button onclick="selectCurve(8)" style="margin:2px;padding:6px 12px;cursor:pointer;">Cochleoid</button>
  <button onclick="selectCurve(9)" style="margin:2px;padding:6px 12px;cursor:pointer;background:#ff6b6b;color:#fff;">Compare All</button>
</div>

<style>
@media print {
  body * { visibility: hidden; }
  #canvas-wrap4, #canvas-wrap4 * { visibility: visible; }
  #canvas-wrap4 {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    background: #fff;
  }
  #canvas-wrap4 canvas { max-width: 100vw; max-height: 100vh; width: auto; height: auto; }
}
#canvas-wrap4 {
  position: relative;
  display: inline-block;
  cursor: pointer;
  border: 1px solid #333;
  border-radius: 8px;
  overflow: hidden;
  line-height: 0;
}
#canvas-wrap4.fullscreen {
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
#canvas-wrap4 .close-btn { display: none; }
#canvas-wrap4.fullscreen .close-btn {
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

<div id="canvas-wrap4">
  <span class="close-btn">&times;</span>
  <canvas id="canvas4"></canvas>
</div>
<br/>
<label for="paramSlider4">Growth rate: <span id="paramVal4">0.50</span></label>
<input type="range" id="paramSlider4" min="5" max="95" value="50" style="width:100%;" />

<script>
(function(){
const wrap = document.getElementById('canvas-wrap4');
const canvas = document.getElementById('canvas4');
const ctx = canvas.getContext('2d');
const slider = document.getElementById('paramSlider4');
const paramVal = document.getElementById('paramVal4');

let W, H, cx, cy, scale, time = 0, animId;
let curveIdx = 0;
let compareMode = false;

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
  scale = Math.min(W, H) * 0.32;
}

const spirals = [
  {
    name: 'Archimedean Spiral', math: 'Archimedes, c. 225 BCE',
    eq: function(t, k) {
      const a = 0.05 + k * 0.25;
      return { x: a * t * Math.cos(t), y: a * t * Math.sin(t) };
    },
    desc: 'Constant spacing between turns. Found in coiled rope and vinyl records.',
    color: '#ff6b6b'
  },
  {
    name: 'Logarithmic Spiral', math: 'Descartes & Bernoulli',
    eq: function(t, k) {
      const b = 0.05 + k * 0.25;
      const r = Math.exp(b * t);
      return { x: r * Math.cos(t), y: r * Math.sin(t) };
    },
    desc: 'Self-similar — looks the same at every scale. The nautilus shell spiral.',
    color: '#4ecdc4'
  },
  {
    name: 'Hyperbolic Spiral', math: 'Pierre Varignon, 1704',
    eq: function(t, k) {
      const a = 0.2 + k * 1.5;
      return { x: a * Math.cos(t) / t, y: a * Math.sin(t) / t };
    },
    desc: 'Winds inward from infinity to a central point. The inverse of the Archimedean.',
    color: '#f9ca24'
  },
  {
    name: "Fermat's Spiral", math: 'Pierre de Fermat, 1636',
    eq: function(t, k) {
      const a = 0.08 + k * 0.3;
      const r = Math.sqrt(t);
      return { x: a * r * Math.cos(t), y: a * r * Math.sin(t) };
    },
    desc: 'Parabolic spiral — the radial distance grows as √t. Two symmetric arms.',
    color: '#a29bfe'
  },
  {
    name: 'Lituus', math: 'Colin Maclaurin, 1722',
    eq: function(t, k) {
      const a = 0.2 + k * 2;
      return { x: a * Math.cos(t) / Math.sqrt(t), y: a * Math.sin(t) / Math.sqrt(t) };
    },
    desc: 'Named after a Roman staff. Winds infinitely around the origin but never reaches it.',
    color: '#fd79a8'
  },
  {
    name: 'Cornu Spiral (Clothoid)', math: 'Marie Alfred Cornu, 1874',
    eq: function(t, k) {
      const s = (t - Math.PI) * (2 + k * 5);
      // Fresnel integral approximation using Simpson-like approach
      let cx = 0, cy = 0;
      const N = 200;
      const ds = s / N;
      for (let i = 0; i < N; i++) {
        const u = i * ds;
        cx += Math.cos(u * u * 0.5) * ds;
        cy += Math.sin(u * u * 0.5) * ds;
      }
      return { x: cx, y: cy };
    },
    desc: 'Curvature changes linearly with length. Used to design highway off-ramps.',
    color: '#00b894'
  },
  {
    name: 'Fibonacci Spiral', math: 'Leonardo Fibonacci, c. 1200',
    eq: function(t, k) {
      const phi = 1.618033988749895;
      const b = Math.log(phi) / (Math.PI / 2);
      const a = 0.1 + k * 0.3;
      const r = a * Math.exp(b * t);
      return { x: r * Math.cos(t), y: r * Math.sin(t) };
    },
    desc: 'Approximated by quarter-circle arcs in Fibonacci squares. A logarithmic spiral with φ.',
    color: '#e17055'
  },
  {
    name: "Poinsot's Spiral", math: 'Louis Poinsot, 19th c.',
    eq: function(t, k) {
      const a = 0.15 + k * 1.5;
      return {
        x: a * Math.cos(t) / (t + 0.5),
        y: a * Math.sin(t) / (t + 0.5)
      };
    },
    desc: 'A generalisation of the hyperbolic spiral. Named after the mechanics pioneer.',
    color: '#74b9ff'
  },
  {
    name: 'Cochleoid', math: 'Roger Cotes, 1722',
    eq: function(t, k) {
      const a = 0.3 + k * 2;
      const sinc = t < 0.001 ? 1 : Math.sin(t) / t;
      return {
        x: a * sinc * Math.cos(t),
        y: a * sinc * Math.sin(t)
      };
    },
    desc: 'Snail-like spiral. "Cochlea" is Latin for snail shell.',
    color: '#ffeaa7'
  }
];

function selectCurve(idx) {
  if (idx === 9) { compareMode = !compareMode; return; }
  compareMode = false;
  curveIdx = idx;
}

function drawSpiral(cx0, cy0, sc, spiral, tMax, alpha) {
  const N = 800;
  const k = slider.value / 100;
  ctx.beginPath();
  let first = true;
  for (let i = 0; i <= N; i++) {
    const t = 0.2 + (i / N) * tMax;
    try {
      const pt = spiral.eq(t, k);
      if (!isFinite(pt.x) || !isFinite(pt.y)) { first = true; continue; }
      const sx = cx0 + pt.x * sc;
      const sy = cy0 - pt.y * sc;
      if (Math.abs(sx - cx) > W || Math.abs(sy - cy) > H) { first = true; continue; }
      if (first) { ctx.moveTo(sx, sy); first = false; }
      else { ctx.lineTo(sx, sy); }
    } catch(e) { first = true; }
  }
  ctx.strokeStyle = spiral.color;
  ctx.globalAlpha = alpha;
  ctx.lineWidth = 2;
  ctx.shadowColor = spiral.color;
  ctx.shadowBlur = 4;
  ctx.stroke();
  ctx.shadowBlur = 0;
  ctx.globalAlpha = 1;
}

function draw() {
  ctx.clearRect(0, 0, W, H);

  const k = slider.value / 100;

  if (compareMode) {
    // Mini view: show all 9 spirals in a grid
    const cols = 3, rows = 3;
    const cellW = W / cols, cellH = H / rows;
    for (let i = 0; i < 9; i++) {
      const col = i % cols, row = Math.floor(i / cols);
      const cxi = cellW * col + cellW / 2;
      const cyi = cellH * row + cellH / 2;
      const sci = Math.min(cellW, cellH) * 0.3;
      drawSpiral(cxi, cyi, sci, spirals[i], Math.PI * 5, 1);
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 10px "Segoe UI", sans-serif';
      ctx.textAlign = 'center';
      ctx.fillText(spirals[i].name, cxi, cyi - cellH * 0.32);
    }
  } else {
    // Single spiral view
    const spiral = spirals[curveIdx];
    drawSpiral(cx, cy, scale, spiral, Math.PI * 7, 1);

    // Tracer
    const tMax = Math.PI * 7;
    const traceT = 0.2 + ((time * 0.04) % 1) * (tMax - 0.2);
    try {
      const dp = spiral.eq(traceT, k);
      if (isFinite(dp.x) && isFinite(dp.y)) {
        const dx = cx + dp.x * scale;
        const dy = cy - dp.y * scale;
        ctx.beginPath();
        ctx.arc(dx, dy, 5, 0, Math.PI * 2);
        ctx.fillStyle = '#fff';
        ctx.fill();
        ctx.strokeStyle = spiral.color;
        ctx.stroke();
      }
    } catch(e) {}

    // Labels
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 15px "Segoe UI", sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText(spiral.name, cx, 18);
    ctx.font = '11px "Segoe UI", sans-serif';
    ctx.fillStyle = '#aaa';
    ctx.fillText(spiral.math, cx, 34);
    ctx.fillText(spiral.desc, cx, H - 10);
  }

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

## How to Tell Your Spirals Apart

All spirals wind around a central point, but their **spacing** is the telltale sign:

| Spiral | Radial equation \\(r(\theta)\\) | Spacing behaviour |
|---|---|---|
| Archimedean | \\(a\theta\\) | Constant — each turn is the same distance from the last |
| Logarithmic | \\(ae^{b\theta}\\) | Grows exponentially — wider and wider |
| Hyperbolic | \\(a/\theta\\) | Shrinks as it winds outward from a starting angle |
| Fermat | \\(a\sqrt{\theta}\\) | Grows like a square root — tighter than Archimedean |
| Lituus | \\(a/\sqrt{\theta}\\) | Compresses as angle increases |

## The Archimedean Spiral: The Practical One

Archimedes described his spiral in *On Spirals* (c. 225 BCE), using it to square the circle and trisect angles. Today, it's everywhere: coiled springs, vinyl record grooves, and scroll compressors all use constant-pitch spirals. The distance between successive windings is exactly \\(2\pi a\\) — always the same.

## The Logarithmic Spiral: Nature's Favourite

Descartes first described it, but Jacob Bernoulli made it famous. He called it *spira mirabilis* — the marvellous spiral — because it's **self-similar**: zoom in or out by any factor, and the curve looks exactly the same. This is why the nautilus shell, rams' horns, spiral galaxies, and even the approach path of a hawk hunting prey all approximate logarithmic spirals. Bernoulli wanted one on his gravestone — but the mason carved an Archimedean spiral by mistake.

## The Cornu Spiral: Saving Lives on the Highway

The Cornu spiral (also called the clothoid or Euler spiral) has curvature that increases linearly with arc length. It's the mathematical basis for **highway transition curves** — the graceful entry and exit of freeway ramps. When you turn a steering wheel at a constant rate, your car traces a Cornu spiral. Before clothoids were used in railway design in the 19th century, trains had to slow dramatically for curves; the smooth transition allowed much higher speeds.

Its parametric equations involve the **Fresnel integrals**:

\\[C(t) = \int_0^t \cos\left(\frac{\pi u^2}{2}\right) du, \quad S(t) = \int_0^t \sin\left(\frac{\pi u^2}{2}\right) du\\]

These integrals have no closed form in elementary functions — a reminder that some of the most practical curves resist simple formulas.

## Fibonacci: The Celebrity Spiral

The Fibonacci spiral is really a logarithmic spiral with growth factor \\(\phi = (1+\sqrt{5})/2 \approx 1.618\\). It's constructed by drawing quarter-circles inside squares whose side lengths follow the Fibonacci sequence. It appears (sometimes genuinely, sometimes wishfully) in sunflowers, pinecones, and the Parthenon. One thing is certain: \\(\phi\\) appears wherever optimal packing meets angular growth.

---

## Nielsen's Spiral: The One That Got Away

Niels Nielsen (1865–1931) discovered a spiral based on special functions that resists simple parameterisation. Unlike the others in our gallery, it cannot be expressed with elementary functions alone — it involves integral representations related to Bessel functions. Nielsen made fundamental contributions to the theory of the gamma function and generalised hypergeometric series; his spiral is a footnote, but a reminder that not every beautiful curve yields to a tidy formula.

---

**Try it:** Click each spiral button to see it alone. Hit **Compare All** for a grid view. The slider adjusts the growth rate — see how each spiral's character changes.

<style>
.curve-selector button { transition: all 0.2s; }
.curve-selector button:hover { transform: scale(1.05); }
</style>
