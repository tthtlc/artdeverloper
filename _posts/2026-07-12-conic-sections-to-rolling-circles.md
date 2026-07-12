---
layout: post
title: "From Conic Sections to Rolling Circles: Eight Curves That Changed Geometry"
tags:
  - conic-sections
  - epicycloid
  - hypocycloid
  - algebraic-curves
  - interactive
  - animation
date: 2026-07-12
math: true
---

Every curve tells a story. Some were carved into sand by Greek geometers slicing cones with planes; others emerged from the Renaissance obsession with rolling wheels. This post brings eight foundational curves to life — from the ancient circle to the heart-shaped cardioid — with animations that reveal how one form can morph into another.

<div class="curve-selector" style="margin: 1em 0;">
  <button onclick="selectCurve(0)" style="margin:2px;padding:6px 12px;cursor:pointer;">Circle</button>
  <button onclick="selectCurve(1)" style="margin:2px;padding:6px 12px;cursor:pointer;">Ellipse</button>
  <button onclick="selectCurve(2)" style="margin:2px;padding:6px 12px;cursor:pointer;">Parabola</button>
  <button onclick="selectCurve(3)" style="margin:2px;padding:6px 12px;cursor:pointer;">Hyperbola</button>
  <button onclick="selectCurve(4)" style="margin:2px;padding:6px 12px;cursor:pointer;">Astroid</button>
  <button onclick="selectCurve(5)" style="margin:2px;padding:6px 12px;cursor:pointer;">Deltoid</button>
  <button onclick="selectCurve(6)" style="margin:2px;padding:6px 12px;cursor:pointer;">Nephroid</button>
  <button onclick="selectCurve(7)" style="margin:2px;padding:6px 12px;cursor:pointer;">Cardioid</button>
  <button onclick="selectCurve(8)" style="margin:2px;padding:6px 12px;cursor:pointer;background:#ff6b6b;color:#fff;">Morph All</button>
</div>

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
#canvas-wrap .close-btn { display: none; }
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
<br/>
<label for="paramSlider">Parameter <em>k</em>: <span id="paramVal">1.00</span></label>
<input type="range" id="paramSlider" min="0" max="100" value="50" style="width:100%;" />

<script>
const wrap = document.getElementById('canvas-wrap');
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const slider = document.getElementById('paramSlider');
const paramVal = document.getElementById('paramVal');

let W, H, cx, cy, scale, time = 0, animId;
let curveIdx = 0;
let morphTarget = 0;
let morphProgress = 1.0;
let isMorphing = false;

function size() {
  const rect = wrap.getBoundingClientRect();
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

// Curve definitions: each returns {x, y} for parameter t in [0, 2π]
const curves = [
  { name: 'Circle', math: 'Ancient Greeks', eq: (t, k) => ({ x: Math.cos(t), y: Math.sin(t) }) },
  { name: 'Ellipse', math: 'Apollonius', eq: (t, k) => ({ x: 1.6 * Math.cos(t), y: Math.sin(t) }) },
  { name: 'Parabola', math: 'Apollonius', eq: (t, k) => {
    const s = (t / Math.PI - 1) * 2.5;
    return { x: s, y: s * s * 0.6 - 0.8 };
  }},
  { name: 'Hyperbola', math: 'Apollonius', eq: (t, k) => {
    const s = (t / Math.PI - 1) * 1.8;
    return { x: 1 / Math.cos(s) * 0.4, y: Math.tan(s) * 0.5 };
  }},
  { name: 'Astroid', math: 'Bernoulli', eq: (t, k) => ({ x: Math.cos(t) ** 3, y: Math.sin(t) ** 3 }) },
  { name: 'Deltoid', math: 'Euler', eq: (t, k) => ({
    x: (2 * Math.cos(t) + Math.cos(2 * t)) / 3,
    y: (2 * Math.sin(t) - Math.sin(2 * t)) / 3
  })},
  { name: 'Nephroid', math: 'Huygens', eq: (t, k) => ({
    x: (3 * Math.cos(t) - Math.cos(3 * t)) / 4,
    y: (3 * Math.sin(t) - Math.sin(3 * t)) / 4
  })},
  { name: 'Cardioid', math: 'Castillon', eq: (t, k) => ({
    x: (2 * Math.cos(t) - Math.cos(2 * t)) / 3,
    y: (2 * Math.sin(t) - Math.sin(2 * t)) / 3
  })},
];

function lerp(a, b, t) { return a + (b - a) * t; }

function getPoint(t, ci) {
  const c = curves[ci];
  const k = slider.value / 100;
  try {
    return c.eq(t, k);
  } catch(e) {
    return { x: 0, y: 0 };
  }
}

function selectCurve(idx) {
  if (idx === 8) {
    // Morph all mode
    isMorphing = true;
    morphTarget = (curveIdx + 1) % 8;
    morphProgress = 0;
  } else {
    isMorphing = false;
    curveIdx = idx;
    morphProgress = 1;
  }
}

function draw() {
  ctx.clearRect(0, 0, W, H);

  // draw subtle grid
  ctx.strokeStyle = 'rgba(255,255,255,0.08)';
  ctx.lineWidth = 0.5;
  for (let i = -5; i <= 5; i++) {
    const x = cx + i * scale * 0.4;
    ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, H); ctx.stroke();
    const y = cy + i * scale * 0.4;
    ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
  }

  // axes
  ctx.strokeStyle = 'rgba(255,255,255,0.2)';
  ctx.lineWidth = 1;
  ctx.beginPath(); ctx.moveTo(0, cy); ctx.lineTo(W, cy); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(cx, 0); ctx.lineTo(cx, H); ctx.stroke();

  if (isMorphing) {
    morphProgress += 0.005;
    if (morphProgress >= 1) {
      morphProgress = 1;
      curveIdx = morphTarget;
      isMorphing = false;
    }
  }

  // Draw animated tracer dot
  const N = 600;
  const traceT = time * 0.02;

  // Draw the curve
  ctx.beginPath();
  let firstPoint = true;
  for (let i = 0; i <= N; i++) {
    const t = (i / N) * Math.PI * 2;
    let pt;
    if (isMorphing) {
      const pA = getPoint(t, curveIdx);
      const pB = getPoint(t, morphTarget);
      pt = {
        x: lerp(pA.x, pB.x, morphProgress),
        y: lerp(pA.y, pB.y, morphProgress)
      };
    } else {
      pt = getPoint(t, curveIdx);
    }
    const sx = cx + pt.x * scale;
    const sy = cy - pt.y * scale;
    if (firstPoint) { ctx.moveTo(sx, sy); firstPoint = false; }
    else { ctx.lineTo(sx, sy); }
  }
  ctx.strokeStyle = '#ff6b6b';
  ctx.lineWidth = 2.5;
  ctx.shadowColor = '#ff6b6b';
  ctx.shadowBlur = 8;
  ctx.stroke();
  ctx.shadowBlur = 0;

  // Draw tracer dot
  let dotPt;
  if (isMorphing) {
    const pA = getPoint(traceT, curveIdx);
    const pB = getPoint(traceT, morphTarget);
    dotPt = { x: lerp(pA.x, pB.x, morphProgress), y: lerp(pA.y, pB.y, morphProgress) };
  } else {
    dotPt = getPoint(traceT, curveIdx);
  }
  const dx = cx + dotPt.x * scale;
  const dy = cy - dotPt.y * scale;
  ctx.beginPath();
  ctx.arc(dx, dy, 6, 0, Math.PI * 2);
  ctx.fillStyle = '#fff';
  ctx.fill();
  ctx.strokeStyle = '#ff6b6b';
  ctx.lineWidth = 2;
  ctx.stroke();

  // Label
  const displayCurve = isMorphing
    ? curves[curveIdx].name + ' → ' + curves[morphTarget].name
    : curves[curveIdx].name;
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 16px "Segoe UI", sans-serif';
  ctx.textAlign = 'center';
  ctx.fillText(displayCurve, cx, 24);
  ctx.font = '12px "Segoe UI", sans-serif';
  ctx.fillStyle = '#aaa';
  const displayMath = isMorphing ? 'morphing...' : curves[curveIdx].math;
  ctx.fillText(displayMath, cx, 42);

  // Param value
  const k = slider.value / 100;
  paramVal.textContent = k.toFixed(2);

  time++;
  animId = requestAnimationFrame(draw);
}

wrap.addEventListener('click', function(e) {
  if (e.target === wrap || e.target === canvas) {
    if (document.fullscreenElement) {
      document.exitFullscreen();
    } else {
      wrap.requestFullscreen();
    }
  }
});
wrap.querySelector('.close-btn').addEventListener('click', function(e) {
  e.stopPropagation();
  if (document.fullscreenElement) document.exitFullscreen();
});
document.addEventListener('fullscreenchange', function() {
  if (document.fullscreenElement === wrap) {
    wrap.classList.add('fullscreen');
  } else {
    wrap.classList.remove('fullscreen');
  }
  setTimeout(size, 100);
});

window.addEventListener('resize', size);
size();
draw();
</script>

---

## 1. Circle — The Mother of All Curves

The circle is where geometry begins. Defined by \\(x = a \cos t, y = a \sin t\\), every point is equidistant from the centre. Ancient cultures from Babylon to Egypt knew its properties, but the Greeks made it the foundation of their cosmos — planets moved in circles because the circle was *perfect*.

In our animation, the circle is the "identity" curve from which the others deviate. Watch how a single parameter transforms it.

## 2. Ellipse — The Stretched Circle

Apollonius of Perga (c. 262–190 BCE) discovered that slicing a cone at a shallow angle produces an ellipse. Parametrically, \\(x = a \cos t, y = b \sin t\\) — just a circle with unequal axes.

Kepler would later show that planets move in ellipses, not circles, breaking a two-thousand-year-old belief. The ellipse is literally the shape of revolution.

## 3. Parabola — The Throw Curve

Every projectile traces a parabola: \\(y = x^2\\). Apollonius named it from the Greek *parabolē* ("application"), and Galileo later proved that cannonballs follow parabolic arcs (ignoring air resistance). It's the only conic section with a single focus — a property exploited by satellite dishes and telescope mirrors everywhere.

## 4. Hyperbola — The Asymptotic Twin

The hyperbola (\\(x = a \sec t, y = b \tan t\\)) is the conic section that escapes to infinity. It has two disconnected branches and two foci. Its asymptotes form an "X" that the curve approaches but never touches — a shape that appears in the shadow of a lampshade on a wall and in the paths of comets that visit the solar system only once.

## 5. Astroid — The Four-Cusped Star

Jump forward to the Bernoulli family: the astroid (\\(x = a \cos^3 t, y = a \sin^3 t\\)) is a hypocycloid with four cusps. Imagine a small circle rolling inside a larger circle of four times its radius — a point on the smaller circle traces this star-like shape. Its name comes from the Greek *astron* (star), and it appears in the shape of certain gear mechanisms.

## 6. Deltoid — The Three-Cusped Curve

When the rolling circle has one-third the radius of the fixed circle, you get a deltoid: \\(x = 2a \cos t + a \cos 2t, y = 2a \sin t - a \sin 2t\\). Euler studied it extensively. It has exactly three cusps and the curious property that any tangent line to the deltoid intersects it at a point whose distance along the tangent to the cusp is constant.

## 7. Nephroid — The Kidney Curve

Huygens discovered the nephroid (\\(x = a(3\cos t - \cos 3t), y = a(3\sin t - \sin 3t)\\)), whose name comes from the Greek *nephros* (kidney). It's the epicycloid formed when the rolling circle has half the radius of the fixed circle. You can see it in your morning coffee: light reflecting off the inside of a cylindrical cup forms a nephroid caustic.

## 8. Cardioid — The Heart of Mathematics

The cardioid (\\(x = a(2\cos t - \cos 2t), y = a(2\sin t - \sin 2t)\\)) is the epicycloid where both circles have equal radii. First studied by Castillon in 1741, it's the shape of the Mandelbrot set's main bulb and the pickup pattern of certain microphones. Its name — "heart-like" — needs no explanation. It is also the envelope of circles passing through a fixed point on a given circle.

---

**Try it:** Click the curve buttons above to switch between shapes, or hit **Morph All** to watch a continuous transformation through the entire family. Drag the slider to vary each curve's internal parameter.

<style>
.curve-selector button { transition: all 0.2s; }
.curve-selector button:hover { transform: scale(1.05); }
</style>
