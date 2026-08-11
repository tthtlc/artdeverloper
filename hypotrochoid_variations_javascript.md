# Hypotrochoid Explorations — 10 Variations and JavaScript Version

This document proposes 10 distinct variations on a hypotrochoid exploration and includes a self-contained JavaScript implementation in Markdown-friendly form. The base curve uses the standard hypotrochoid parametric equations with parameters for outer radius \(R\), inner radius \(r\), and pen offset \(d\).[page:1]

## 10 variation ideas

1. **Breathing offset** — animate the pen offset `d` with a slow sine wave so the curve appears to inhale and exhale over time while preserving the underlying hypotrochoid structure.[page:1]
2. **Ratio morphing** — smoothly interpolate the radius ratio `R/r` between nearby rational values to create transitions between closed and near-closed forms.[page:1]
3. **Phase-stacked ribbons** — draw several hypotrochoids with tiny phase offsets and low opacity to produce woven ribbon bands.[page:1]
4. **Hue-by-curvature** — color the line by local turning rate or curvature so cusps and flatter lobes read as different thermal zones.
5. **Audio-reactive offset** — map microphone amplitude or FFT bands to `d`, stroke width, or opacity to turn the drawing into a music visualizer.
6. **Noise-distorted trace** — perturb the point position with a gentle coherent noise field to create organic, less mechanical guilloché forms.
7. **Elliptic drift mode** — exploit the special family near `R = 2r` to generate ellipse-like or Tusi-couple-adjacent motifs, then slowly drift parameters away from that regime.[page:1]
8. **Polar bloom** — convert sampled hypotrochoid points into particles that expand radially after being traced, leaving a flower-like echo.
9. **Mirror kaleidoscope** — render one curve segment and replicate it through rotational or bilateral symmetry for mandala compositions.
10. **Trail engraving** — accumulate many passes with multiply/screen compositing and tiny random jitter to imitate engraved security-print or banknote textures, echoing spirograph-style guilloché aesthetics.[web:6][page:1]

## JavaScript version

Save the following as `hypotrochoid-variations.html` or paste it into any Markdown renderer that preserves fenced code blocks.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Hypotrochoid Variations</title>
  <style>
    :root {
      color-scheme: dark;
      --bg: #0f1115;
      --panel: #171a21;
      --text: #e8ecf3;
      --muted: #9aa4b2;
      --accent: #67e8f9;
      --border: rgba(255,255,255,0.12);
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Inter, system-ui, sans-serif;
      background: radial-gradient(circle at top, #1b2130 0%, var(--bg) 50%);
      color: var(--text);
      min-height: 100vh;
      display: grid;
      grid-template-columns: 320px 1fr;
    }
    aside {
      border-right: 1px solid var(--border);
      background: rgba(23,26,33,0.85);
      backdrop-filter: blur(10px);
      padding: 20px;
      overflow: auto;
    }
    main {
      display: grid;
      grid-template-rows: 1fr auto;
      min-width: 0;
    }
    .canvas-wrap {
      display: grid;
      place-items: center;
      padding: 18px;
    }
    canvas {
      width: min(100%, 1000px);
      height: min(100%, 1000px);
      aspect-ratio: 1;
      border-radius: 18px;
      background: #0b0d12;
      border: 1px solid var(--border);
      box-shadow: 0 20px 60px rgba(0,0,0,0.35);
    }
    h1 { margin: 0 0 8px; font-size: 1.4rem; }
    p { color: var(--muted); line-height: 1.5; }
    .controls { display: grid; gap: 14px; margin-top: 18px; }
    .row { display: grid; gap: 6px; }
    .row label { font-size: 0.92rem; }
    .row output { color: var(--accent); font-size: 0.85rem; }
    input[type="range"], select, button {
      width: 100%;
    }
    select, button {
      background: var(--panel);
      color: var(--text);
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 10px 12px;
    }
    button { cursor: pointer; }
    .toolbar {
      display: flex;
      gap: 10px;
      padding: 14px 18px 18px;
      flex-wrap: wrap;
      border-top: 1px solid var(--border);
      background: rgba(12,14,18,0.7);
    }
    .note { font-size: 0.85rem; color: var(--muted); margin-top: 14px; }
    @media (max-width: 900px) {
      body { grid-template-columns: 1fr; grid-template-rows: auto 1fr; }
      aside { border-right: 0; border-bottom: 1px solid var(--border); }
    }
  </style>
</head>
<body>
  <aside>
    <h1>Hypotrochoid Variations</h1>
    <p>Interactive canvas for exploring 10 stylistic variations based on the classic hypotrochoid equation.</p>

    <div class="controls">
      <div class="row">
        <label for="mode">Variation</label>
        <select id="mode">
          <option value="classic">1. Classic</option>
          <option value="breathing">2. Breathing offset</option>
          <option value="ratioMorph">3. Ratio morphing</option>
          <option value="phaseStack">4. Phase-stacked ribbons</option>
          <option value="curvatureHue">5. Hue-by-curvature</option>
          <option value="noise">6. Noise-distorted trace</option>
          <option value="ellipse">7. Elliptic drift</option>
          <option value="bloom">8. Polar bloom</option>
          <option value="mirror">9. Mirror kaleidoscope</option>
          <option value="engrave">10. Trail engraving</option>
        </select>
      </div>

      <div class="row">
        <label for="R">Outer radius R</label>
        <input id="R" type="range" min="60" max="260" value="180" />
        <output id="RVal">180</output>
      </div>

      <div class="row">
        <label for="r">Inner radius r</label>
        <input id="r" type="range" min="10" max="180" value="63" />
        <output id="rVal">63</output>
      </div>

      <div class="row">
        <label for="d">Offset d</label>
        <input id="d" type="range" min="10" max="220" value="92" />
        <output id="dVal">92</output>
      </div>

      <div class="row">
        <label for="turns">Turns</label>
        <input id="turns" type="range" min="3" max="60" value="18" />
        <output id="turnsVal">18</output>
      </div>

      <div class="row">
        <label for="samples">Samples</label>
        <input id="samples" type="range" min="400" max="8000" value="2600" step="100" />
        <output id="samplesVal">2600</output>
      </div>

      <div class="row">
        <label for="speed">Animation speed</label>
        <input id="speed" type="range" min="0" max="200" value="35" />
        <output id="speedVal">35</output>
      </div>
    </div>

    <p class="note">Tip: try `R=150`, `r=56`, `d=110` for floral loops, or move toward `R≈2r` for ellipse-like families.</p>
  </aside>

  <main>
    <div class="canvas-wrap">
      <canvas id="canvas" width="1200" height="1200"></canvas>
    </div>
    <div class="toolbar">
      <button id="toggle">Pause</button>
      <button id="randomize">Randomize</button>
      <button id="clear">Clear trails</button>
    </div>
  </main>

  <script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const controls = {
      mode: document.getElementById('mode'),
      R: document.getElementById('R'),
      r: document.getElementById('r'),
      d: document.getElementById('d'),
      turns: document.getElementById('turns'),
      samples: document.getElementById('samples'),
      speed: document.getElementById('speed')
    };
    const outputs = {
      R: document.getElementById('RVal'),
      r: document.getElementById('rVal'),
      d: document.getElementById('dVal'),
      turns: document.getElementById('turnsVal'),
      samples: document.getElementById('samplesVal'),
      speed: document.getElementById('speedVal')
    };

    let playing = true;
    let time = 0;
    let persistentFade = 0.09;

    function syncOutputs() {
      for (const k of ['R','r','d','turns','samples','speed']) outputs[k].value = controls[k].value;
    }
    syncOutputs();
    Object.values(controls).forEach(el => el.addEventListener('input', syncOutputs));

    function gcd(a, b) {
      a = Math.abs(Math.round(a));
      b = Math.abs(Math.round(b));
      while (b) [a, b] = [b, a % b];
      return a || 1;
    }

    function hypotrochoidPoint(t, R, r, d, phase = 0) {
      return {
        x: (R - r) * Math.cos(t + phase) + d * Math.cos(((R - r) / r) * (t + phase)),
        y: (R - r) * Math.sin(t + phase) - d * Math.sin(((R - r) / r) * (t + phase))
      };
    }

    function pseudoNoise(x) {
      return Math.sin(x * 1.37) * 0.5 + Math.sin(x * 0.23 + 1.7) * 0.35 + Math.sin(x * 2.91 + 0.2) * 0.15;
    }

    function getParams() {
      let R = +controls.R.value;
      let r = Math.max(1, +controls.r.value);
      let d = +controls.d.value;
      let turns = +controls.turns.value;
      let samples = +controls.samples.value;
      let speed = +controls.speed.value / 1000;
      const mode = controls.mode.value;

      if (mode === 'breathing') d *= 0.72 + 0.28 * (1 + Math.sin(time * 1.3)) / 2;
      if (mode === 'ratioMorph') r = Math.max(8, r + 10 * Math.sin(time * 0.7));
      if (mode === 'ellipse') R = 2 * r + 8 * Math.sin(time * 0.5);

      return { R, r, d, turns, samples, speed, mode };
    }

    function drawBackground(alpha = 1) {
      ctx.save();
      ctx.globalAlpha = alpha;
      const g = ctx.createRadialGradient(600, 560, 40, 600, 600, 700);
      g.addColorStop(0, '#101521');
      g.addColorStop(1, '#06070a');
      ctx.fillStyle = g;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.restore();
    }

    function drawCurve(points, style = {}) {
      if (!points.length) return;
      ctx.save();
      ctx.lineWidth = style.lineWidth ?? 1.5;
      ctx.strokeStyle = style.strokeStyle ?? '#67e8f9';
      ctx.globalAlpha = style.alpha ?? 1;
      ctx.beginPath();
      ctx.moveTo(points[0].x, points[0].y);
      for (let i = 1; i < points.length; i++) ctx.lineTo(points[i].x, points[i].y);
      ctx.stroke();
      ctx.restore();
    }

    function buildPoints(params, extraPhase = 0) {
      const { R, r, d, turns, samples, mode } = params;
      const pts = [];
      const closeTurns = R / gcd(R, r);
      const totalTurns = Math.max(turns, closeTurns);
      const tMax = Math.PI * 2 * totalTurns;

      for (let i = 0; i <= samples; i++) {
        const t = (i / samples) * tMax;
        let p = hypotrochoidPoint(t, R, r, d, extraPhase);

        if (mode === 'noise') {
          const n1 = pseudoNoise(t * 0.9 + time * 0.8);
          const n2 = pseudoNoise(t * 0.7 - time * 0.6 + 4);
          p.x += n1 * 12;
          p.y += n2 * 12;
        }

        if (mode === 'bloom') {
          const amp = 1 + 0.12 * Math.sin(t * 8 + time * 2.2);
          p.x *= amp;
          p.y *= amp;
        }

        pts.push({ x: 600 + p.x * 2.2, y: 600 + p.y * 2.2, t });
      }
      return pts;
    }

    function render() {
      if (playing) time += getParams().speed;

      const params = getParams();
      const { mode } = params;

      if (mode === 'engrave') {
        ctx.fillStyle = `rgba(6,7,10,${persistentFade})`;
        ctx.fillRect(0, 0, canvas.width, canvas.height);
      } else {
        drawBackground(1);
      }

      if (mode === 'phaseStack') {
        for (let k = 0; k < 8; k++) {
          const pts = buildPoints(params, k * 0.06 + time * 0.08);
          drawCurve(pts, { strokeStyle: `hsla(${180 + k * 12}, 90%, 70%, 0.16)`, lineWidth: 1.4 });
        }
      } else if (mode === 'mirror') {
        const base = buildPoints(params, time * 0.1);
        for (let m = 0; m < 6; m++) {
          const a = (Math.PI * 2 * m) / 6;
          const ca = Math.cos(a), sa = Math.sin(a);
          const pts = base.map(p => {
            const x = p.x - 600, y = p.y - 600;
            return { x: 600 + x * ca - y * sa, y: 600 + x * sa + y * ca };
          });
          drawCurve(pts, { strokeStyle: `hsla(${m * 45 + 180}, 90%, 72%, 0.24)`, lineWidth: 1.2 });
        }
      } else if (mode === 'curvatureHue') {
        const pts = buildPoints(params, time * 0.08);
        for (let i = 1; i < pts.length - 1; i++) {
          const a = pts[i - 1], b = pts[i], c = pts[i + 1];
          const dx1 = b.x - a.x, dy1 = b.y - a.y;
          const dx2 = c.x - b.x, dy2 = c.y - b.y;
          const ang1 = Math.atan2(dy1, dx1), ang2 = Math.atan2(dy2, dx2);
          let delta = Math.abs(ang2 - ang1);
          if (delta > Math.PI) delta = 2 * Math.PI - delta;
          const hue = 180 + delta * 300;
          ctx.strokeStyle = `hsla(${hue}, 90%, 70%, 0.9)`;
          ctx.lineWidth = 1.6;
          ctx.beginPath();
          ctx.moveTo(a.x, a.y);
          ctx.lineTo(b.x, b.y);
          ctx.stroke();
        }
      } else if (mode === 'engrave') {
        for (let i = 0; i < 12; i++) {
          const jitter = (i - 6) * 0.003;
          const pts = buildPoints({ ...params, d: params.d * (1 + jitter) }, time * 0.05 + i * 0.01);
          drawCurve(pts, { strokeStyle: `rgba(230,240,255,0.06)`, lineWidth: 0.9 });
        }
      } else {
        const pts = buildPoints(params, time * 0.12);
        drawCurve(pts, { strokeStyle: '#7dd3fc', lineWidth: 1.8, alpha: 0.95 });
      }

      requestAnimationFrame(render);
    }

    document.getElementById('toggle').addEventListener('click', e => {
      playing = !playing;
      e.target.textContent = playing ? 'Pause' : 'Play';
    });

    document.getElementById('randomize').addEventListener('click', () => {
      controls.R.value = 100 + Math.floor(Math.random() * 120);
      controls.r.value = 24 + Math.floor(Math.random() * 90);
      controls.d.value = 25 + Math.floor(Math.random() * 140);
      controls.turns.value = 8 + Math.floor(Math.random() * 24);
      controls.samples.value = 1600 + Math.floor(Math.random() * 3600);
      controls.mode.selectedIndex = Math.floor(Math.random() * controls.mode.options.length);
      syncOutputs();
    });

    document.getElementById('clear').addEventListener('click', () => drawBackground(1));

    drawBackground(1);
    render();
  </script>
</body>
</html>
```

## Notes

The implementation uses the standard hypotrochoid equations for sampling points on a canvas, then layers animation and rendering strategies on top of that mathematical core.[page:1] Closed-form behavior depends on the radius relationship between `R` and `r`, which is why rational ratios tend to produce repeating forms and nearby perturbations produce slowly drifting patterns.[page:1][web:5]
