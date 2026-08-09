---
layout: post
title: "Twelve Curves That Secretly Rule Geometry — From the Infinity Symbol to the Modern Skyscraper"
date: 2026-08-09
categories:
  - Geometry
  - Mathematics
tags:
  - algebraic-curves
  - lemniscate
  - witch-of-agnesi
  - folium-of-descartes
  - superellipse
  - cassini-oval
  - interactive
  - history-of-mathematics
share: true
read_time: true
excerpt: "From the infinity symbol to the shape of Stockholm's most famous roundabout, twelve algebraic curves reveal how a single polynomial equation can birth shapes of extraordinary beauty. Interactive sliders let you vary each curve's defining parameter and watch the transformation unfold in real time."
math: true
---

**Challenge to the reader:** Using the interactive widget below, find the parameter value where the Cassini oval pinches into a lemniscate — the infinity symbol. Then, explain why the Folium of Descartes must cross itself at exactly one point, no matter how you vary its parameter. (Answers at the end of the post.)

Between the 17th and 19th centuries, mathematicians discovered a menagerie of curves that defied intuition: loops that cross themselves, shapes that resemble leaves and witches' hats, and one curve so versatile it gave us the modern office building. These twelve algebraic curves — each defined by a single polynomial equation in $x$ and $y$ — reveal how the search for geometric beauty drove centuries of discovery. More remarkably, many of them turn out to be special cases of one another, connected by a hidden algebraic web.

**Why this matters:** Algebraic curves are not museum pieces. The superellipse shapes the furniture you sit on and the roundabouts you drive through. The lemniscate appears in elliptic function theory, cryptography, and the symbol for infinity itself. Understanding these twelve curves is a tour through the engine room of analytic geometry — and the interactive sliders below let you turn the knobs yourself.

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
	window.selectCurve = selectCurve;
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

## 1. The Ancient Toolkit: Curves That Solved Impossible Problems

Long before Descartes fused algebra with geometry in 1637, Greek geometers were already using curves to crack the three classical construction problems: doubling the cube, trisecting an angle, and squaring the circle. They couldn't solve them with compass and straightedge alone — so they invented curves that could.

**The Cissoid of Diocles** (c. 180 BCE) was designed to double the cube. Given a cube of volume $V$, Diocles showed how his cissoid — whose name means "ivy-shaped" — could construct the edge length of a cube with volume $2V$. The curve is defined by the property that a line through the origin cuts it at a point whose distances satisfy a precise ratio. Its cusp at the origin makes it one of the earliest examples of a singular point on an otherwise smooth curve.

**The Conchoid of Nicomedes** (c. 200 BCE) cracked angle trisection. Nicomedes described it as a "shell-shaped" curve (conchoid = "mussel-like"). Fix a line and a point not on it; the conchoid is the locus of points at a fixed distance from the line, measured along rays through the fixed point. By constructing a specific conchoid, any angle can be trisected — a problem that stumped geometers using only straightedge and compass.

**The Kampyle of Eudoxus** (c. 350 BCE) is the oldest curve in our collection. Its name means "bent" or "curved" in Greek — and it looks exactly like a flexed bow. Eudoxus, the father of the method of exhaustion (a precursor to integration), used it in his astronomical models. With equation $x^4 = a^2(x^2 + y^2)$, it's a quartic curve that was cutting-edge mathematics two millennia before calculus.

**Challenge:** Why does the cissoid have a cusp at the origin? Trace the parametric path as $t \to 0$ and explain geometrically what happens to the tangent direction. Hint: compute $\lim_{t \to 0} dy/dx$ from the parametric form.

---

## 2. The Baroque Garden: Curves of the 17th and 18th Centuries

The invention of analytic geometry by Descartes and Fermat in the 1630s unleashed a flood of new curves. Every polynomial equation in $x$ and $y$ became fair game — and mathematicians competed to find the strangest, most beautiful specimens.

**The Folium of Descartes** (1638). Descartes challenged Fermat to find the tangent to $x^3 + y^3 = 3axy$. Fermat succeeded, humiliating Descartes — who retaliated by claiming Fermat's method wasn't general enough. The resulting "leaf" has a self-intersection at the origin, and its area was one of the first non-trivial successes of integral calculus. Move the slider to see the loop swell and the asymptote shift.

**The Lemniscate of Bernoulli** (1694). Jacob Bernoulli discovered this figure-eight curve while studying elastic deformations. Its defining property is delightfully simple: the product of distances from any point on the curve to two fixed foci is constant. When this constant equals the square of half the focal distance, the oval pinches at the waist — becoming the lemniscate, a special case of the Cassini oval. Bernoulli was so enchanted that he had it engraved on his tombstone with the inscription *"Eadem mutata resurgo"* — "Though changed, I shall arise the same."

**The Witch of Agnesi** (1748). Maria Gaetana Agnesi published the first comprehensive calculus textbook by a woman — *Instituzioni analitiche* — in 1748. The curve $y = a^3/(x^2 + a^2)$ appears as a gentle bell-shaped hump, which Agnesi called *versiera*. When John Colson translated her work into English, he mistook *versiera* for *avversiera* ("witch"). The name stuck — a permanent monument to a translation error, and to Agnesi's remarkable achievement as one of the first women to publish original mathematics.

**Challenge:** The lemniscate's polar equation is $r^2 = 2a^2 \cos 2\theta$. Verify that this is equivalent to the Cartesian form $(x^2 + y^2)^2 = 2a^2(x^2 - y^2)$ by substituting $x = r\cos\theta$, $y = r\sin\theta$. Then use the polar form to explain why the lemniscate only exists for $\theta \in [-\pi/4, \pi/4] \cup [3\pi/4, 5\pi/4]$.

---

## 3. The Analytic Turn: Newton's Classification and Its Offspring

In 1701, Isaac Newton classified all cubic curves into 72 types — a monumental work of systematic geometry. Several of our curves emerged from this classification project.

**The Serpentine Curve.** Newton named this one himself. With equation $x^2 y + y = a x$, it snakes through the plane in a graceful S-shape. It's a cubic with an asymptote at $x = 0$ — one of the simplest curves to exhibit both a turning point and an infinite branch. The slider lets you stretch its serpentine coil and watch how the inflection point shifts.

**The Cruciform.** This cross-shaped curve, known to medieval geometers but formalised later, comes from the deceptively simple equation $x^2 y^2 = a^2(x^2 + y^2)$. Its four arms extend to infinity along the axes, making it one of the few algebraic curves with asymptotes in all four cardinal directions. Despite its dramatic appearance, it is symmetric under both $x \to -x$ and $y \to -y$.

**The Bullet Nose Curve.** A 19th-century favourite of artillery engineers, this curve mimics the ogive profile of a projectile — the shape that minimises air resistance at subsonic speeds. Its equation involves a cusp and a vertical asymptote, producing the characteristic blunt-tipped, gracefully tapering silhouette that you still see on rockets and high-speed trains today.

**The Kite Curve.** Related to the astroid (hypocycloid of four cusps) but with independently scaled axes, this curve resembles a diamond kite. Its parametric form $x = a\cos^3 t$, $y = b\sin^3 t$ generalises the astroid by letting $a$ and $b$ differ — a simple tweak that turns a star-like shape into something entirely different.

---

## 4. The Modern Metamorphosis: One Equation, Infinite Shapes

**The Lamé Superellipse** (1818). Gabriel Lamé generalised the ellipse by replacing the exponent $2$ with an arbitrary real number $n$:

$$
\left\lvert \frac{x}{a} \right\rvert^{\,n} + \left\lvert \frac{y}{b} \right\rvert^{\,n} = 1
$$

When $n = 2$, you get an ordinary ellipse. For $n > 2$, the shape bulges outward toward a rectangle. For $n < 1$, it pinches inward to a star-like four-pointed shape. For $n = 1$, it's a diamond. Lamé could not have predicted that, 140 years later, the Danish polymath Piet Hein would use the superellipse ($n = 2.5$) to design Stockholm's Sergels Torg roundabout — and that the "superegg," a three-dimensional superellipse, would become a bestselling novelty item because it can stand upright on either end, apparently defying gravity.

**The Cassini Oval** (1680). Giovanni Domenico Cassini, the astronomer who discovered the gap in Saturn's rings, proposed that planets move along these ovals rather than Keplerian ellipses. He was wrong about planetary orbits — but his curves turned out to be mathematically richer than ellipses. The Cassini oval is the locus of points whose *product* of distances to two fixed foci is constant (compare: an ellipse uses the *sum*). As the slider moves, you see the oval morph through three topological regimes: a single loop, a pinched figure-eight (the lemniscate!), and two separate loops. One family, three distinct topological phases — a miniature lesson in bifurcation theory.

**Challenge:** At what critical parameter value does the Cassini oval transition from a single loop to two separate ovals? Express your answer in terms of the focal half-distance $c$ and the constant product $a^2$. Then verify your prediction using the interactive slider — does the visual match the algebra?

---

## 5. How the Twelve Curves Connect

One of the deepest insights of algebraic geometry is that apparently unrelated curves are often connected by transformations, specialisations, or inversions.

| Curve | Equation Type | Degree | Key Property | Related To |
|---|---|---|---|---|
| Kampyle | Implicit | 4 | Bow-shaped; oldest in collection | Conchoid (shared polar structure) |
| Cissoid | Implicit | 3 | Cusp at origin; cube-doubling | Witch (inversion in a circle) |
| Conchoid | Implicit | 4 | Angle trisection; polar form | Kampyle |
| Lemniscate | Implicit | 4 | Product of distances = constant | Cassini (degenerate case), elliptic functions |
| Folium | Implicit | 3 | Self-intersecting loop; first calculus area | — |
| Witch | Rational | 3 | Bell-shaped; mistranslated name | Cissoid (inversion) |
| Serpentine | Implicit | 3 | S-shaped; Newton-classified | — |
| Cruciform | Implicit | 4 | Four infinite arms; asymptotes on axes | — |
| Bullet Nose | Implicit | 3 | Cusp + asymptote; ogive profile | — |
| Kite | Parametric | — | Scaled astroid; diamond shape | Astroid, Superellipse |
| Superellipse | Implicit | $n$-dependent | One family: star → diamond → circle → square | Lamé curve family, Piet Hein design |
| Cassini Oval | Implicit | 4 | Three topological regimes in one family | Lemniscate (special case $a = c$) |

The most striking connection: the Witch of Agnesi and the Cissoid of Diocles are *inverses* of each other with respect to a suitably chosen circle. Two curves, separated by two millennia and discovered for entirely different purposes, turn out to be geometric mirror images under inversion.

---

## 6. Deeper Significance: Why These Curves Still Matter

These twelve curves are not dusty antiques. They encode a profound lesson about mathematics itself: **simple polynomial equations in two variables generate an astonishing variety of topological behaviours.** The same equation family that gives you a smooth oval can, with a small parameter tweak, produce a self-intersecting loop or a pair of disconnected lobes. This sensitivity — technically, a *bifurcation* — is the same phenomenon that appears in dynamical systems, fluid mechanics, and population biology.

The lemniscate reappears in the theory of elliptic functions and elliptic curves — the same elliptic curves that underpin modern cryptography (ECC) and were central to Andrew Wiles's proof of Fermat's Last Theorem. The superellipse taught designers that a single continuous parameter can interpolate between radically different aesthetic regimes — a principle now embedded in parametric CAD software used to design everything from cars to coffee cups.

And the Cassini oval? Its three-regime topology (one loop → pinched lemniscate → two loops) is a textbook example of a *phase transition* in a two-parameter family — the same mathematical structure that describes liquid–gas transitions in thermodynamics. When you drag the Cassini slider and watch the oval tear itself in two, you're watching the geometric twin of water boiling into steam.

**Use the slider** to vary each curve's defining parameter and watch the shape transform. Click **Auto-Tour** to sit back and let the curves introduce themselves in sequence.

---

## 7. Final Challenge

**Synthesis challenge:** Choose any two curves from the collection. Prove that one is the *inverse* of the other with respect to a suitably chosen circle. (Hint: the Witch of Agnesi and the Cissoid of Diocles are inverses of each other. Start by writing both in polar form and applying the inversion transformation $r \to 1/r$. You'll also need to translate the centre of inversion to the right point.)

Then, using the interactive widget, find the parameter value where the Cassini oval becomes the lemniscate. Compute the product of distances from any point on the lemniscate to the two foci — verify that it matches the defining constant $c^2$, where $c$ is the focal half-distance. Does your slider observation match the algebraic condition $a = c$?

---

**Answers to opening challenges:**

- **Cassini → lemniscate:** The transition occurs when the constant product $a^2$ equals $c^2$ (the square of the focal half-distance). At this value, the oval pinches to a figure-eight — the lemniscate of Bernoulli.
- **Folium self-intersection:** The Folium's equation $x^3 + y^3 = 3axy$ is symmetric under swapping $x$ and $y$. At the origin $(0,0)$, the curve approaches along two distinct branches (one with slope $0$, one with infinite slope), forcing a self-intersection regardless of the parameter $a$.

<style>
.curve-selector button { transition: all 0.2s; }
.curve-selector button:hover { transform: scale(1.05); }
</style>
