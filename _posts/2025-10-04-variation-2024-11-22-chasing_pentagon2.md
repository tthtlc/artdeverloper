```markdown
---
layout: fullscreen
title: "Waveform Lotus: Generative Oscillating Petals"
tags:
  - graphics
---

<canvas id="lotusCanvas" width="600" height="600" style="width:100vw; height:100vh; display:block; background:#161616"></canvas>
<script>
const canvas = document.getElementById('lotusCanvas');
const ctx = canvas.getContext('2d');
const width = canvas.width;
const height = canvas.height;
let time = 0;

function hsv2rgb(h, s, v) {
    let f = (n, k = (h / 60 + n) % 6) =>
      v - v * s * Math.max(Math.min(k, 4 - k, 1), 0);
    return [f(5), f(3), f(1)];
}

// Lotus parameters
const NUM_PETALS = 14;
const PETAL_BASE = 90;
const PETAL_LENGTH = 210;
const PETAL_WAVELENGTH = 3.3;
const PETAL_PHASE = Math.PI / 3;
const PETAL_MOTION_SPEED = 2.9;
const PETAL_FILL_ALPHA = 0.36;
const PETAL_STROKE_ALPHA = 0.73;

// Draw a single petal as an oscillating Bezier curve fan
function drawPetal(cx, cy, angle, t, idx) {
    ctx.save();
    ctx.translate(cx, cy);
    ctx.rotate(angle);

    // Color cycling for psychedelic effect
    const hue = (360 * idx / NUM_PETALS + t*27) % 360;
    const rgb = hsv2rgb(hue, 0.78, 1);
    ctx.strokeStyle = `rgba(${(rgb[0]*255).toFixed()},${(rgb[1]*255).toFixed()},${(rgb[2]*255).toFixed()},${PETAL_STROKE_ALPHA})`;
    ctx.lineWidth = 2;
    ctx.beginPath();

    // The petal's shape in polar coordinates, modulated by time and sine wave 
    ctx.moveTo(0, 0);

    let points = [];
    const steps = 54;
    for(let i=0; i<=steps; i++) {
        let theta = -Math.PI / 7 + (i/steps)*Math.PI / 3.5;
        // Main petal curve radius
        let base = PETAL_BASE + PETAL_LENGTH * Math.sin(theta);
        // Add a time-modulated waveform
        let wave = 18 * Math.sin(PETAL_WAVELENGTH * theta + t*PETAL_MOTION_SPEED + PETAL_PHASE*idx)
                 + 11 * Math.cos((PETAL_WAVELENGTH*theta + t*1.2 + idx*0.4));
        let r = base + wave;

        let x = r * Math.cos(theta);
        let y = r * Math.sin(theta);
        points.push([x, y]);
        ctx.lineTo(x, y);
    }

    ctx.closePath();
    // Glow effect: fill with more transparency for inner auras
    ctx.globalAlpha = PETAL_FILL_ALPHA;
    ctx.fillStyle = `rgba(${(rgb[0]*255).toFixed()},${(rgb[1]*255).toFixed()},${(rgb[2]*255).toFixed()},1)`;
    ctx.fill();
    ctx.globalAlpha = 1.0;
    ctx.stroke();

    // Draw inner secondary highlight on each petal
    ctx.beginPath();
    ctx.moveTo(0, 0);
    for(let i=0; i<points.length; i+=4) {
        let [x, y] = points[i];
        ctx.lineTo(0.83*x, 0.83*y);
    }
    ctx.closePath();
    ctx.strokeStyle = `rgba(255,255,255,0.10)`;
    ctx.lineWidth = 1;
    ctx.stroke();

    ctx.restore();
}

// Animate
function draw() {
    ctx.clearRect(0, 0, width, height);

    // Draw a glowing central disk
    let center_color = hsv2rgb((time*22)%360, 0.38, 1);
    let grad = ctx.createRadialGradient(width/2, height/2, 0, width/2, height/2, 80 + 10*Math.sin(time*0.9));
    grad.addColorStop(0, `rgba(${(center_color[0]*255).toFixed()},${(center_color[1]*255).toFixed()},${(center_color[2]*255).toFixed()},0.73)`);
    grad.addColorStop(1, 'rgba(0,0,0,0)');
    ctx.beginPath();
    ctx.arc(width/2, height/2, 85 + 7*Math.cos(time*1.2), 0, 2*Math.PI);
    ctx.fillStyle = grad;
    ctx.fill();

    // Petals
    for(let i=0; i<NUM_PETALS; i++) {
        drawPetal(
            width/2, height/2,
            (2*Math.PI/NUM_PETALS)*i + 0.3*Math.sin(time*0.2+i),
            time,
            i
        );
    }

    // Secondary subtle outer flickering rings
    for(let j=0; j<3; j++) {
        let radius = 245 + 18*Math.sin(time*1.2 + j*1.9);
        ctx.save();
        ctx.globalAlpha = 0.18 - 0.04*j;
        ctx.strokeStyle = `hsl(${Math.floor(280 + 33 * Math.sin(time*0.6 + j))},70%,70%)`;
        ctx.lineWidth = 3-j;
        ctx.beginPath();
        ctx.arc(width/2, height/2, radius, 0, 2*Math.PI);
        ctx.stroke();
        ctx.restore();
    }

    time += 0.018;
    requestAnimationFrame(draw);
}

// For retina screens: resize and scale canvas
function adjustCanvasSize() {
    let dpr = window.devicePixelRatio || 1;
    canvas.width = window.innerWidth * dpr;
    canvas.height = window.innerHeight * dpr;
    ctx.setTransform(1,0,0,1,0,0);
    ctx.scale(dpr, dpr);
}
window.addEventListener('resize', () => {
    adjustCanvasSize();
});
adjustCanvasSize();
draw();
</script>
```
