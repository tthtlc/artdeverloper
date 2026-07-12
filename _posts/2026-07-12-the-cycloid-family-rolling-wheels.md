---
layout: post
title: "The Cycloid Family: When Circles Roll, Mathematics Unfolds"
tags:
  - cycloid
  - epicycloid
  - hypocycloid
  - catenary
  - brachistochrone
  - interactive
  - animation
date: 2026-07-12
math: true
---

What shape does a point on a rolling wheel trace? The answer — the cycloid — sparked one of the most bitter rivalries in mathematical history. But the cycloid is just the beginning. Roll a circle inside another circle, outside it, or along a line with different radii, and a whole family of curves emerges, each with its own surprising properties.

<div class="curve-selector" style="margin: 1em 0;">
  <button onclick="selectCurve(0)" style="margin:2px;padding:6px 12px;cursor:pointer;">Cycloid</button>
  <button onclick="selectCurve(1)" style="margin:2px;padding:6px 12px;cursor:pointer;">Prolate</button>
  <button onclick="selectCurve(2)" style="margin:2px;padding:6px 12px;cursor:pointer;">Curtate</button>
  <button onclick="selectCurve(3)" style="margin:2px;padding:6px 12px;cursor:pointer;">Epicycloid</button>
  <button onclick="selectCurve(4)" style="margin:2px;padding:6px 12px;cursor:pointer;">Hypocycloid</button>
  <button onclick="selectCurve(5)" style="margin:2px;padding:6px 12px;cursor:pointer;">Involute</button>
  <button onclick="selectCurve(6)" style="margin:2px;padding:6px 12px;cursor:pointer;">Tractrix</button>
  <button onclick="selectCurve(7)" style="margin:2px;padding:6px 12px;cursor:pointer;">Catenary</button>
  <button onclick="selectCurve(8)" style="margin:2px;padding:6px 12px;cursor:pointer;background:#ff6b6b;color:#fff;">Animate Roll</button>
</div>

<style>
@media print {
  body * { visibility: hidden; }
  #canvas-wrap3, #canvas-wrap3 * { visibility: visible; }
  #canvas-wrap3 {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    background: #fff;
  }
  #canvas-wrap3 canvas { max-width: 100vw; max-height: 100vh; width: auto; height: auto; }
}
#canvas-wrap3 {
  position: relative;
  display: inline-block;
  cursor: pointer;
  border: 1px solid #333;
  border-radius: 8px;
  overflow: hidden;
  line-height: 0;
}
#canvas-wrap3.fullscreen {
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
#canvas-wrap3 .close-btn { display: none; }
#canvas-wrap3.fullscreen .close-btn {
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

<div id="canvas-wrap3">
  <span class="close-btn">&times;</span>
  <canvas id="canvas3"></canvas>
</div>
<br/>
<label for="paramSlider3">Rolling radius ratio: <span id="paramVal3">0.50</span></label>
<input type="range" id="paramSlider3" min="10" max="90" value="50" style="width:100%;" />

<script>
(function(){
const wrap = document.getElementById('canvas-wrap3');
const canvas = document.getElementById('canvas3');
const ctx = canvas.getContext('2d');
const slider = document.getElementById('paramSlider3');
const paramVal = document.getElementById('paramVal3');

let W, H, cx, cy, scale, time = 0, animId;
let curveIdx = 0;
let showRolling = true;

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

function selectCurve(idx) {
  if (idx === 8) { showRolling = !showRolling; return; }
  curveIdx = idx;
}

// The curve definitions with roll data for animation
const curves = [
  {
    name: 'Cycloid',
    math: 'Galileo & Mersenne',
    type: 'line',
    R: 1, r: 0, d: 1,
    desc: 'The brachistochrone: fastest path under gravity.'
  },
  {
    name: 'Prolate Cycloid',
    math: 'Christiaan Huygens',
    type: 'line',
    R: 1, r: 0, d: 1.5,
    desc: 'Tracing point beyond the wheel rim — loops back on itself.'
  },
  {
    name: 'Curtate Cycloid',
    math: 'Christiaan Huygens',
    type: 'line',
    R: 1, r: 0, d: 0.5,
    desc: 'Tracing point inside the wheel — a gentle wave with no cusps.'
  },
  {
    name: 'Epicycloid',
    math: 'Ole Rømer, 1674',
    type: 'epi',
    R: 2, r: 1, d: 1,
    desc: 'Circle rolling on the OUTSIDE of another circle.'
  },
  {
    name: 'Hypocycloid',
    math: 'Ole Rømer, 1674',
    type: 'hypo',
    R: 3, r: 1, d: 1,
    desc: 'Circle rolling on the INSIDE of another — the Spirograph curve.'
  },
  {
    name: 'Involute of Circle',
    math: 'Christiaan Huygens',
    type: 'involute',
    R: 1,
    desc: 'The path traced by unwinding a taut string from a circle — gear tooth profile.'
  },
  {
    name: 'Tractrix',
    math: 'Gottfried Leibniz, 1693',
    type: 'tractrix',
    R: 1,
    desc: 'The "pursuit curve" — dragged by a leash. Revolve it to get the pseudosphere.'
  },
  {
    name: 'Catenary',
    math: 'Huygens, Leibniz, Bernoulli',
    type: 'catenary',
    R: 1,
    desc: 'The shape of a hanging chain. NOT a parabola, as Galileo once thought.'
  }
];

function getCurvePoint(curve, t, k) {
  const c = curve;
  switch (c.type) {
    case 'line': {
      const R = c.R;
      const d = c.d;
      // Adjust d based on slider for prolate/curtate
      const dd = (curveIdx === 0) ? R : (curveIdx === 1 ? 1 + k : 1 - k * 0.9);
      return {
        x: R * (t - dd * Math.sin(t)),
        y: R * (1 - dd * Math.cos(t)),
        cx: R * t, cy: R  // rolling circle center for animation
      };
    }
    case 'epi': {
      const Rr = 1 + k * 3;
      const rr = 1;
      return {
        x: (Rr + rr) * Math.cos(t) - rr * Math.cos((Rr + rr) / rr * t),
        y: (Rr + rr) * Math.sin(t) - rr * Math.sin((Rr + rr) / rr * t),
        cx: (Rr + rr) * Math.cos(t), cy: (Rr + rr) * Math.sin(t)
      };
    }
    case 'hypo': {
      const Rr = 2 + k * 4;
      const rr = 1;
      return {
        x: (Rr - rr) * Math.cos(t) + rr * Math.cos((Rr - rr) / rr * t),
        y: (Rr - rr) * Math.sin(t) - rr * Math.sin((Rr - rr) / rr * t),
        cx: (Rr - rr) * Math.cos(t), cy: (Rr - rr) * Math.sin(t)
      };
    }
    case 'involute': {
      const a = 0.4 + k * 1.2;
      return {
        x: a * (Math.cos(t) + t * Math.sin(t)),
        y: a * (Math.sin(t) - t * Math.cos(t))
      };
    }
    case 'tractrix': {
      const a = 0.5 + k * 1.5;
      return {
        x: a * (t - Math.tanh(t)),
        y: a / Math.cosh(t)
      };
    }
    case 'catenary': {
      const a = 0.3 + k * 1.2;
      return {
        x: t,
        y: a * Math.cosh(t / a)
      };
    }
    default: return { x: Math.cos(t), y: Math.sin(t) };
  }
}

function draw() {
  ctx.clearRect(0, 0, W, H);

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

  const curve = curves[curveIdx];
  const k = slider.value / 100;
  const N = 1000;
  const tMax = (curve.type === 'line') ? Math.PI * 4 : Math.PI * 2;

  // Draw rolling circle / base circle for cycloid types
  if (showRolling && (curve.type === 'line')) {
    const tNow = (time * 0.03) % (Math.PI * 4);
    const pt = getCurvePoint(curve, tNow, k);
    const rcx = cx + pt.cx * scale * 0.6;
    const rcy = cy - pt.cy * scale * 0.6;
    // Base line
    ctx.strokeStyle = 'rgba(255,255,255,0.25)';
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.moveTo(cx - 3 * scale * 0.6, cy - scale * 0.6);
    ctx.lineTo(cx + 8 * scale * 0.6, cy - scale * 0.6);
    ctx.stroke();
    // Rolling circle
    ctx.strokeStyle = 'rgba(255,200,100,0.4)';
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    ctx.arc(rcx, rcy, scale * 0.6, 0, Math.PI * 2);
    ctx.stroke();
    // Spoke
    ctx.strokeStyle = 'rgba(255,200,100,0.3)';
    ctx.beginPath(); ctx.moveTo(rcx, rcy); ctx.lineTo(rcx + pt.x * scale * 0.6 - pt.cx * scale * 0.6, rcy - (pt.y * scale * 0.6 - pt.cy * scale * 0.6)); ctx.stroke();
  }

  if (showRolling && (curve.type === 'epi' || curve.type === 'hypo')) {
    const tNow = (time * 0.02) % (Math.PI * 2);
    const pt = getCurvePoint(curve, tNow, k);
    const R = (curve.type === 'epi') ? (1 + k * 3 + 1) : (2 + k * 4 - 1);
    // Base circle
    ctx.strokeStyle = 'rgba(255,255,255,0.2)';
    ctx.lineWidth = 1;
    ctx.beginPath();
    ctx.arc(cx, cy, R * scale * 0.5, 0, Math.PI * 2);
    ctx.stroke();
    // Rolling circle
    if (pt.cx !== undefined) {
      const rcx2 = cx + pt.cx * scale * 0.5;
      const rcy2 = cy - pt.cy * scale * 0.5;
      ctx.strokeStyle = 'rgba(255,200,100,0.4)';
      ctx.lineWidth = 1.5;
      ctx.beginPath();
      ctx.arc(rcx2, rcy2, scale * 0.5, 0, Math.PI * 2);
      ctx.stroke();
    }
  }

  // Draw the curve
  ctx.beginPath();
  let first = true;
  for (let i = 0; i <= N; i++) {
    const t = (i / N) * tMax;
    try {
      const pt = getCurvePoint(curve, t, k);
      if (!isFinite(pt.x) || !isFinite(pt.y)) { first = true; continue; }
      let sx, sy;
      if (curve.type === 'line') {
        sx = cx + (pt.x - Math.PI * 2) * scale * 0.6;
        sy = cy - pt.y * scale * 0.6;
      } else {
        sx = cx + pt.x * scale * 0.5;
        sy = cy - pt.y * scale * 0.5;
      }
      if (Math.abs(sx - cx) > W || Math.abs(sy - cy) > H) { first = true; continue; }
      if (first) { ctx.moveTo(sx, sy); first = false; }
      else { ctx.lineTo(sx, sy); }
    } catch(e) { first = true; }
  }
  ctx.strokeStyle = '#f9ca24';
  ctx.lineWidth = 2.5;
  ctx.shadowColor = '#f9ca24';
  ctx.shadowBlur = 6;
  ctx.stroke();
  ctx.shadowBlur = 0;

  // Tracer
  const traceT = (time * 0.03) % tMax;
  try {
    const dp = getCurvePoint(curve, traceT, k);
    if (isFinite(dp.x) && isFinite(dp.y)) {
      let ddx, ddy;
      if (curve.type === 'line') {
        ddx = cx + (dp.x - Math.PI * 2) * scale * 0.6;
        ddy = cy - dp.y * scale * 0.6;
      } else {
        ddx = cx + dp.x * scale * 0.5;
        ddy = cy - dp.y * scale * 0.5;
      }
      ctx.beginPath();
      ctx.arc(ddx, ddy, 5, 0, Math.PI * 2);
      ctx.fillStyle = '#fff';
      ctx.fill();
      ctx.strokeStyle = '#f9ca24';
      ctx.stroke();
    }
  } catch(e) {}

  // Labels
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 15px "Segoe UI", sans-serif';
  ctx.textAlign = 'center';
  ctx.fillText(curve.name, cx, 18);
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

## The Cycloid Wars

Galileo first studied the cycloid around 1599 and even tried to determine its area by weighing paper cutouts. Mersenne, Descartes, Fermat, Pascal, and Huygens all worked on it — and each claimed priority. Pascal, in a moment of religious fervour, abandoned mathematics but later, sleepless with a toothache, started thinking about the cycloid and the pain vanished. He took it as a divine sign and returned to math.

The cycloid has an almost magical property: it is the **brachistochrone** — the curve of fastest descent under gravity. Drop a bead from any height along a cycloidal wire, and it reaches the bottom faster than on any other path, including a straight line. Johann Bernoulli posed this as a challenge in 1696; Newton solved it overnight, anonymously — but Bernoulli recognised "the lion by his claw."

## Prolate and Curtate: Variations on a Wheel

Move the tracing point beyond the wheel's rim (prolate) and the curve loops back on itself — like the path of a point on a train wheel flange. Move it inside the rim (curtate) and you get a smooth undulation. All three are given by the same equations:

\\[x = a(t - k \sin t), \quad y = a(1 - k \cos t)\\]

where \\(k = 1\\) is the cycloid, \\(k > 1\\) is prolate, and \\(k < 1\\) is curtate.

## Epicycloids and Hypocycloids: The Spirograph Curves

When one circle rolls around another, you get epicycloids (outside) and hypocycloids (inside). The ratio of radii determines the number of cusps. If the ratio is rational, the curve closes after a finite number of revolutions — the principle behind every Spirograph toy ever sold.

Ole Rømer (better known for measuring the speed of light) first studied these systematically in 1674 while investigating gear teeth profiles. The cardioid, nephroid, astroid, and deltoid from our previous post are all special cases.

## Catenary vs. Parabola: Galileo's Mistake

Galileo thought a hanging chain formed a parabola. It doesn't — it forms a catenary, \\(y = a \cosh(x/a)\\). The difference is subtle but real. When Robert Hooke announced he could determine the ideal shape for an arch by inverting a hanging chain, he was applying the catenary's properties. The Gateway Arch in St. Louis is an inverted catenary.

## The Tractrix: A Dog on a Leash

Imagine pulling a heavy object by a string while walking in a straight line. The path the object traces is a tractrix. Leibniz discovered it in 1693. Revolve it around its asymptote and you get the **pseudosphere** — the first concrete realisation of hyperbolic geometry, where Euclid's parallel postulate fails.

---

**Controls:** Click a curve to select it. Drag the slider to adjust the rolling ratio. Toggle **Animate Roll** to show or hide the rolling wheel. Click the canvas for fullscreen.

<style>
.curve-selector button { transition: all 0.2s; }
.curve-selector button:hover { transform: scale(1.05); }
</style>
