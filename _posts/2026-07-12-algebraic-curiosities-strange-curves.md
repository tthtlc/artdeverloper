---
layout: post
title: "Algebraic Curiosities: Twelve Strange Curves and the Mathematicians Who Found Them"
tags:
  - algebraic-curves
  - lemniscate
  - witch-of-agnesi
  - folium
  - superellipse
  - interactive
  - animation
date: 2026-07-12
math: true
---

Between the 17th and 19th centuries, mathematicians discovered a menagerie of curves that defied intuition: loops that cross themselves, shapes that resemble leaves and witches' hats, and one curve so versatile it gave us the modern office building. These twelve algebraic curves — each defined by a polynomial equation — reveal how the search for geometric beauty drove centuries of discovery.

<div class="curve-selector" style="margin: 1em 0;">
  <button onclick="selectCurve(0)" style="margin:2px;padding:6px 12px;cursor:pointer;">Lemniscate</button>
  <button onclick="selectCurve(1)" style="margin:2px;padding:6px 12px;cursor:pointer;">Folium</button>
  <button onclick="selectCurve(2)" style="margin:2px;padding:6px 12px;cursor:pointer;">Witch of Agnesi</button>
  <button onclick="selectCurve(3)" style="margin:2px;padding:6px 12px;cursor:pointer;">Cissoid</button>
  <button onclick="selectCurve(4)" style="margin:2px;padding:6px 12px;cursor:pointer;">Conchoid</button>
  <button onclick="selectCurve(5)" style="margin:2px;padding:6px 12px;cursor:pointer;">Kampyle</button>
  <button onclick="selectCurve(6)" style="margin:2px;padding:6px 12px;cursor:pointer;">Serpentine</button>
  <button onclick="selectCurve(7)" style="margin:2px;padding:6px 12px;cursor:pointer;">Cruciform</button>
  <button onclick="selectCurve(8)" style="margin:2px;padding:6px 12px;cursor:pointer;">Bullet Nose</button>
  <button onclick="selectCurve(9)" style="margin:2px;padding:6px 12px;cursor:pointer;">Kite</button>
  <button onclick="selectCurve(10)" style="margin:2px;padding:6px 12px;cursor:pointer;">Superellipse</button>
  <button onclick="selectCurve(11)" style="margin:2px;padding:6px 12px;cursor:pointer;">Cassini Oval</button>
  <button onclick="selectCurve(12)" style="margin:2px;padding:6px 12px;cursor:pointer;background:#ff6b6b;color:#fff;">Auto-Tour</button>
</div>

<style>
@media print {
  body * { visibility: hidden; }
  #canvas-wrap2, #canvas-wrap2 * { visibility: visible; }
  #canvas-wrap2 {
    position: absolute;
    top: 0; left: 0; width: 100%; height: 100%;
    display: flex; align-items: center; justify-content: center;
    background: #fff;
  }
  #canvas-wrap2 canvas { max-width: 100vw; max-height: 100vh; width: auto; height: auto; }
}
#canvas-wrap2 {
  position: relative;
  display: inline-block;
  cursor: pointer;
  border: 1px solid #333;
  border-radius: 8px;
  overflow: hidden;
  line-height: 0;
}
#canvas-wrap2.fullscreen {
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
#canvas-wrap2 .close-btn { display: none; }
#canvas-wrap2.fullscreen .close-btn {
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

<div id="canvas-wrap2">
  <span class="close-btn">&times;</span>
  <canvas id="canvas2"></canvas>
</div>
<br/>
<label for="paramSlider2">Variation: <span id="paramVal2">0.50</span></label>
<input type="range" id="paramSlider2" min="5" max="95" value="50" style="width:100%;" />

<script>
(function(){
const wrap = document.getElementById('canvas-wrap2');
const canvas = document.getElementById('canvas2');
const ctx = canvas.getContext('2d');
const slider = document.getElementById('paramSlider2');
const paramVal = document.getElementById('paramVal2');

let W, H, cx, cy, scale, time = 0, animId;
let curveIdx = 0;
let isTouring = false;
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
  scale = Math.min(W, H) * 0.38;
}

const curves = [
  {
    name: 'Lemniscate of Bernoulli',
    math: 'Jacob Bernoulli, 1694',
    eq: function(t, k) {
      const a = 0.7 + k * 0.6;
      const denom = 1 + Math.sin(t) * Math.sin(t);
      return {
        x: a * Math.sqrt(2) * Math.cos(t) / denom,
        y: a * Math.sqrt(2) * Math.sin(t) * Math.cos(t) / denom
      };
    },
    desc: 'The ∞ symbol is a lemniscate. Bernoulli called it the "pendant ribbon."'
  },
  {
    name: 'Folium of Descartes',
    math: 'René Descartes, 1638',
    eq: function(t, k) {
      const a = 0.5 + k * 1.5;
      const t3 = t * t * t;
      const denom = 1 + t3;
      // Avoid singularity
      const safe = Math.abs(denom) < 0.001 ? 0.001 : denom;
      return {
        x: 3 * a * t / safe,
        y: 3 * a * t * t / safe
      };
    },
    range: [-2.5, 2.5],
    desc: 'A leaf-shaped loop — the first algebraic curve whose area was found using calculus.'
  },
  {
    name: 'Witch of Agnesi',
    math: 'Maria Agnesi, 1748',
    eq: function(t, k) {
      const a = 0.3 + k * 1.2;
      return {
        x: a * Math.tan(t),
        y: a * Math.cos(t) * Math.cos(t)
      };
    },
    range: [-1.3, 1.3],
    desc: 'Named after a mistranslation of "versiera." One of the first math books by a woman.'
  },
  {
    name: 'Cissoid of Diocles',
    math: 'Diocles, c. 180 BCE',
    eq: function(t, k) {
      const a = 0.3 + k * 1.2;
      const ct = Math.cos(t);
      if (Math.abs(ct) < 0.01) return { x: 0, y: 0 };
      const s2 = Math.sin(t) * Math.sin(t);
      const tant = Math.tan(t);
      return {
        x: a * s2 / ct,
        y: a * s2 * tant
      };
    },
    range: [-1.2, 1.2],
    desc: 'Used by the ancient Greeks to solve the Delian problem of doubling the cube.'
  },
  {
    name: 'Conchoid of Nicomedes',
    math: 'Nicomedes, c. 200 BCE',
    eq: function(t, k) {
      const a = 0.5;
      const b = k * 2;
      const ct = Math.cos(t);
      const st = Math.sin(t);
      if (Math.abs(st) < 0.01) return { x: 0, y: 0 };
      return {
        x: a * ct / st + b * ct,
        y: a + b * st
      };
    },
    range: [0.15, Math.PI - 0.15],
    desc: 'Nicomedes invented this to trisect angles — one of the three classical problems.'
  },
  {
    name: 'Kampyle of Eudoxus',
    math: 'Eudoxus, c. 350 BCE',
    eq: function(t, k) {
      const a = 0.3 + k * 1.2;
      const ct = Math.cos(t);
      if (Math.abs(ct) < 0.01) return { x: 0, y: 0 };
      return {
        x: a / ct,
        y: a * Math.tan(t) / ct
      };
    },
    range: [-1.3, 1.3],
    desc: 'A curve shaped like a bent bow — "kampyle" is Greek for "curved."'
  },
  {
    name: 'Serpentine Curve',
    math: 'Isaac Newton, 1701',
    eq: function(t, k) {
      const a = 0.5 + k * 1.5;
      return {
        x: a * Math.sin(t) * Math.cos(t),
        y: a * Math.sin(t) * Math.sin(t)
      };
    },
    desc: 'Named for its snake-like S-shape. Newton classified cubic curves into 72 types.'
  },
  {
    name: 'Cruciform',
    math: 'Medieval geometers',
    eq: function(t, k) {
      const a = 0.5 + k * 1.5;
      const ct = Math.cos(t);
      const st = Math.sin(t);
      if (Math.abs(ct) < 0.02) return { x: 0, y: 0 };
      return {
        x: a * Math.tan(t),
        y: a * st
      };
    },
    range: [-1.3, 1.3],
    desc: 'A cross-like shape with four infinite branches. Simple equation, striking form.'
  },
  {
    name: 'Bullet Nose Curve',
    math: '19th-century artillery',
    eq: function(t, k) {
      const a = 0.4 + k * 1.2;
      const ct = Math.cos(t);
      if (Math.abs(ct) < 0.005) return { x: 0, y: 0 };
      return {
        x: a * ct,
        y: Math.abs(Math.sin(t)) / ct * a * 0.5
      };
    },
    range: [-1.3, 1.3],
    desc: 'Resembles the ogive profile of a bullet or rocket nose cone.'
  },
  {
    name: 'Kite Curve',
    math: 'Geometers, 19th c.',
    eq: function(t, k) {
      const a = 0.5 + k * 1.2;
      return {
        x: a * Math.cos(t) * Math.cos(t) * Math.cos(t),
        y: a * Math.sin(t) * Math.sin(t) * Math.sin(t)
      };
    },
    desc: 'Related to the astroid but scaled independently on each axis. Flies like a kite.'
  },
  {
    name: 'Lamé Superellipse',
    math: 'Gabriel Lamé, 1818',
    eq: function(t, k) {
      const a = 1;
      const n = 0.3 + k * 3.7; // exponent controls roundness
      const p = 2 / n;
      const ct = Math.cos(t);
      const st = Math.sin(t);
      return {
        x: a * Math.sign(ct) * Math.pow(Math.abs(ct), p),
        y: a * Math.sign(st) * Math.pow(Math.abs(st), p)
      };
    },
    desc: 'Piet Hein popularised this. From star to square to circle — one family rules them all.'
  },
  {
    name: 'Cassini Oval',
    math: 'Giovanni Cassini, 1680',
    eq: function(t, k) {
      const a = 0.4 + k * 1.2;
      const c = 1.0;
      const r2 = Math.cos(2 * t) * c * c;
      const inner = Math.sqrt(Math.max(0, a * a * a * a - c * c * c * c * Math.sin(2 * t) * Math.sin(2 * t)));
      const r = Math.sqrt(Math.max(0, r2 + inner));
      return {
        x: r * Math.cos(t) * 0.7,
        y: r * Math.sin(t) * 0.7
      };
    },
    desc: 'Cassini thought planets moved along these ovals. The lemniscate is a special case.'
  }
];

function selectCurve(idx) {
  if (idx === 12) {
    isTouring = !isTouring;
    if (!isTouring) return;
    tourTimer = 0;
  } else {
    isTouring = false;
    curveIdx = idx;
  }
}

function draw() {
  ctx.clearRect(0, 0, W, H);

  // Grid
  ctx.strokeStyle = 'rgba(255,255,255,0.06)';
  ctx.lineWidth = 0.5;
  for (let i = -6; i <= 6; i++) {
    const x = cx + i * scale * 0.3;
    ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, H); ctx.stroke();
    const y = cy + i * scale * 0.3;
    ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
  }
  ctx.strokeStyle = 'rgba(255,255,255,0.15)';
  ctx.lineWidth = 1;
  ctx.beginPath(); ctx.moveTo(0, cy); ctx.lineTo(W, cy); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(cx, 0); ctx.lineTo(cx, H); ctx.stroke();

  if (isTouring) {
    tourTimer++;
    if (tourTimer > 240) {
      tourTimer = 0;
      curveIdx = (curveIdx + 1) % 12;
    }
  }

  const curve = curves[curveIdx];
  const tRange = curve.range || [0, Math.PI * 2];
  const N = 800;
  const k = slider.value / 100;

  // Draw curve
  ctx.beginPath();
  let first = true;
  for (let i = 0; i <= N; i++) {
    const t = tRange[0] + (i / N) * (tRange[1] - tRange[0]);
    try {
      const pt = curve.eq(t, k);
      if (!isFinite(pt.x) || !isFinite(pt.y)) { first = true; continue; }
      const sx = cx + pt.x * scale;
      const sy = cy - pt.y * scale;
      if (Math.abs(sx - cx) > W * 1.5 || Math.abs(sy - cy) > H * 1.5) { first = true; continue; }
      if (first) { ctx.moveTo(sx, sy); first = false; }
      else { ctx.lineTo(sx, sy); }
    } catch(e) { first = true; }
  }
  ctx.strokeStyle = '#4ecdc4';
  ctx.lineWidth = 2.5;
  ctx.shadowColor = '#4ecdc4';
  ctx.shadowBlur = 6;
  ctx.stroke();
  ctx.shadowBlur = 0;

  // Tracer dot
  const traceT = tRange[0] + ((time * 0.02) % 1) * (tRange[1] - tRange[0]);
  try {
    const dp = curve.eq(traceT, k);
    if (isFinite(dp.x) && isFinite(dp.y)) {
      const dx = cx + dp.x * scale;
      const dy = cy - dp.y * scale;
      ctx.beginPath();
      ctx.arc(dx, dy, 5, 0, Math.PI * 2);
      ctx.fillStyle = '#fff';
      ctx.fill();
      ctx.strokeStyle = '#4ecdc4';
      ctx.lineWidth = 2;
      ctx.stroke();
    }
  } catch(e) {}

  // Labels
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 15px "Segoe UI", sans-serif';
  ctx.textAlign = 'center';
  const label = isTouring ? 'Auto-Tour: ' + curve.name : curve.name;
  ctx.fillText(label, cx, 20);
  ctx.font = '11px "Segoe UI", sans-serif';
  ctx.fillStyle = '#aaa';
  ctx.fillText(curve.math, cx, 38);
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

## The Lemniscate: Symbol of Infinity

Jacob Bernoulli discovered the lemniscate in 1694 while studying elastic curves. Its figure-eight shape became the symbol for infinity (∞). The equation \\((x^2 + y^2)^2 = 2a^2(x^2 - y^2)\\) produces a curve where the product of distances to two fixed points is constant — making it a special case of the Cassini oval.

Bernoulli was so enchanted by it that he had it engraved on his tombstone with the inscription *"Eadem mutata resurgo"* — "Though changed, I shall arise the same."

## The Folium: Descartes' Leaf

In 1638, Descartes challenged Fermat to find the tangent to \\(x^3 + y^3 = 3axy\\). Fermat succeeded, humiliating Descartes — who retaliated by claiming Fermat's method wasn't general enough. The resulting "leaf" has a self-intersection at the origin, and its area was one of the first non-trivial successes of integral calculus.

## The Witch That Wasn't

Maria Gaetana Agnesi published the first comprehensive calculus textbook by a woman in 1748. The curve \\(y = a^3/(x^2 + a^2)\\) was called *versiera* in Italian. When John Colson translated her work into English, he mistook *versiera* for *avversiera* ("witch"). The name stuck — a permanent monument to a translation error, and to Agnesi's remarkable achievement.

## The Superellipse: From Math to Architecture

Gabriel Lamé generalised the ellipse in 1818: \\(|x/a|^n + |y/b|^n = 1\\). For \\(n = 2\\) you get an ellipse; for larger \\(n\\), the shape becomes increasingly rectangular. In 1959, Piet Hein applied the superellipse to urban design — the Sergels Torg roundabout in Stockholm is a superellipse. He also designed the "superegg," a three-dimensional superellipse that stands upright on a flat surface, defying intuition.

**Use the slider** to vary each curve's defining parameter and watch the shape transform. Click **Auto-Tour** to sit back and let the curves introduce themselves.

<style>
.curve-selector button { transition: all 0.2s; }
.curve-selector button:hover { transform: scale(1.05); }
</style>
