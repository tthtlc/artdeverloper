---
layout: fullscreen
title: "Hypnotic Lissajous Starfields"
tags:
  - graphics
---

<style>
canvas {
    background-color: #101018;
    border: 1px solid #333;
    margin-bottom: 20px;
}
.controls {
    display: flex;
    flex-direction: column;
    align-items: center;
}
.controls label {
    margin-right: 10px;
}
.controls input {
    margin-bottom: 10px;
}
button {
    margin-top: 10px;
    padding: 8px 16px;
    font-size: 16px;
    cursor: pointer;
}
</style>
<script src="https://cdn.jsdelivr.net/npm/gif.js/dist/gif.js"></script>
<canvas id="animationCanvas" width="600" height="600"></canvas>
<div class="controls">
    <div>
        <label for="lissA">Lissajous a:</label>
        <input type="range" id="lissA" name="lissA" min="2" max="9" step="1" value="5">
        <span id="lissAValue">5</span>
    </div>
    <div>
        <label for="lissB">Lissajous b:</label>
        <input type="range" id="lissB" name="lissB" min="2" max="9" step="1" value="4">
        <span id="lissBValue">4</span>
    </div>
    <div>
        <label for="starCount">Stars per ring:</label>
        <input type="range" id="starCount" name="starCount" min="5" max="50" step="1" value="18">
        <span id="starCountValue">18</span>
    </div>
    <div>
        <label for="rings">Number of rings:</label>
        <input type="range" id="rings" name="rings" min="2" max="10" step="1" value="5">
        <span id="ringsValue">5</span>
    </div>
    <button id="captureBtn">Capture JPG</button>
    <button id="generateGif">Generate GIF</button>
</div>

<script>
// --- Parameters and DOM ---
const canvas = document.getElementById('animationCanvas');
const ctx = canvas.getContext('2d');
const centerX = canvas.width/2;
const centerY = canvas.height/2;

let lissA = 5;
let lissB = 4;
let starCount = 18;
let rings = 5;
let phase = 0;

const lissASlider = document.getElementById('lissA');
const lissAValueDisplay = document.getElementById('lissAValue');
const lissBSlider = document.getElementById('lissB');
const lissBValueDisplay = document.getElementById('lissBValue');
const starCountSlider = document.getElementById('starCount');
const starCountValueDisplay = document.getElementById('starCountValue');
const ringsSlider = document.getElementById('rings');
const ringsValueDisplay = document.getElementById('ringsValue');

// Update control displays and listen for user input
lissASlider.addEventListener('input', (e) => {
    lissA = parseInt(e.target.value);
    lissAValueDisplay.textContent = lissA;
});
lissBSlider.addEventListener('input', (e) => {
    lissB = parseInt(e.target.value);
    lissBValueDisplay.textContent = lissB;
});
starCountSlider.addEventListener('input', (e) => {
    starCount = parseInt(e.target.value);
    starCountValueDisplay.textContent = starCount;
});
ringsSlider.addEventListener('input', (e) => {
    rings = parseInt(e.target.value);
    ringsValueDisplay.textContent = rings;
});

// --- GIF Setup ---
const gif = new GIF({
    workers: 2,
    workerScript: "/assets/js/gif.worker.js",
    quality: 10
});
let frameCount = 0;
let maxFrames = 120;

// --- Utility for psychedelic palette ---
function hsl(hu, s=90, l=60, a=1) {
    return `hsla(${(hu%360)},${s}%,${l}%,${a})`;
}

// --- Lissajous starfield generator ---
function getLissajousPoints(cx, cy, baseR, ringIdx, stars, time, a, b, phaseOffset=0) {
    const points = [];
    // Each ring has a base radius, slightly pulsating
    const t = time/40 + ringIdx*0.2;
    const ringPhase = phaseOffset + ringIdx * Math.PI/10 + Math.sin(t*0.7 + ringIdx)*0.2;
    const r = baseR + 18 * Math.sin(t + ringIdx) + 6 * Math.sin((t+ringIdx)*2);
    for (let i=0; i<stars; i++) {
        let theta = 2*Math.PI*i/stars;
        // Lissajous formula for point offset, mapped to polar angle
        // The *shape* evolves as phase and ringIdx changes
        const x = cx + r*Math.sin(a*theta + time/80 + ringPhase);
        const y = cy + r*Math.sin(b*theta + time/60 + ringPhase+Math.PI/2);
        points.push({x, y, theta});
    }
    return points;
}

// Draw an abstract glowing star
function drawStar(x, y, theta, points=5, inner=0.21, outer=1.0, size=8, color='#fff', glowColor='#ff0', alpha=1) {
    ctx.save();
    ctx.globalAlpha = alpha*0.8;
    ctx.translate(x, y);
    ctx.rotate(theta);
    // Glow
    ctx.shadowColor = glowColor;
    ctx.shadowBlur = 16;
    // Star path
    ctx.beginPath();
    for (let i=0; i<points*2; i++) {
        const angle = Math.PI*i/(points);
        const rad = (i%2===0) ? outer*size : inner*size;
        ctx.lineTo(Math.cos(angle)*rad, Math.sin(angle)*rad);
    }
    ctx.closePath();
    ctx.fillStyle = color;
    ctx.fill();
    ctx.shadowBlur = 0;
    ctx.restore();
}

// Draw iridescent lines between two sets of points
function drawConnections(pointsA, pointsB, time, ringIdx) {
    for (let i=0; i<pointsA.length; i++) {
        let pa = pointsA[i];
        let pb = pointsB[i];
        const hue = 220 + Math.sin(time/36 + i*0.07 + ringIdx)*100 + ringIdx*25;
        ctx.save();
        ctx.beginPath();
        ctx.moveTo(pa.x, pa.y);
        ctx.lineTo(pb.x, pb.y);
        ctx.strokeStyle = hsl(hue, 90, 60, 0.78);
        ctx.lineWidth = 1.2 + 0.5*Math.sin(i+time/45 + ringIdx);
        ctx.shadowColor = hsl(hue, 100, 60, 1);
        ctx.shadowBlur = 8;
        ctx.stroke();
        ctx.restore();
    }
}

// --- Animation ---
function clearCanvasWithBackground() {
    // Slightly translucent background (to create "echoes" for higher trippiness!)
    ctx.globalAlpha = 0.17;
    ctx.fillStyle = "#101018";
    ctx.fillRect(0,0,canvas.width,canvas.height);
    ctx.globalAlpha = 1;
}

function animate() {
    clearCanvasWithBackground();

    const time = phase;
    // For each ring, get positions
    let prevPoints = null;
    let baseRadius = 80;
    for (let r=0; r<rings; r++) {
        const points = getLissajousPoints(
            centerX, centerY,
            baseRadius + r*42, r,
            starCount,
            time + r*24,
            lissA + ((r%2) ? 1 : 0),
            lissB + ((r%2) ? 0 : 1),
            r*0.2
        );
        // Draw connection lines to previous ring
        if (prevPoints) {
            drawConnections(prevPoints, points, time, r);
        }
        // Draw the stars for this ring
        for (let i=0; i<points.length; i++) {
            // Color cycles per star and ring
            const starHue = ( (i*360/starCount) + time + r*40 ) % 360;
            drawStar(
                points[i].x, points[i].y,
                Math.PI/4 + Math.sin(points[i].theta + time/15 + r)*0.5,
                5 + (r%2),
                0.22+0.12*Math.sin(i+time/420+r),
                1,
                9 + 5*Math.sin(r+time/50 + i/6),
                hsl(starHue,95,starHue<180?74:58,0.97),
                hsl(starHue+90,95,68,0.9),
                0.80
            );
        }
        prevPoints = points;
    }

    // Central pulse (final star)
    drawStar(centerX, centerY, time/90, 8, 0.15, 1, 16 + 8*Math.sin(time/30), hsl(phase*2,96,78,0.7), hsl(phase*2+120,100,54,1), 0.85 + 0.12*Math.sin(time/13));

    // Animation phase control
    phase += 2.3;

    if (frameCount < maxFrames) {
        gif.addFrame(canvas, {copy: true, delay: 60});
        frameCount++;
    }

    requestAnimationFrame(animate);
}

// --- Capture JPG ---
document.getElementById('captureBtn').addEventListener('click', function() {
    // Opaque background
    ctx.globalAlpha = 1;
    ctx.fillStyle = "#101018";
    ctx.fillRect(0, 0, canvas.width, canvas.height);

    // Re-draw a still frame
    let tempPhase = phase;
    phase = 0;
    animate();
    phase = tempPhase;

    setTimeout(function() {
        const dataURL = canvas.toDataURL('image/jpeg', 1.0);
        const link = document.createElement('a');
        link.href = dataURL;
        link.download = 'lissajous_starfields.jpg';
        link.click();
    }, 80);
});

// --- Generate GIF ---
document.getElementById('generateGif').addEventListener('click', function() {
    gif.on('finished', function(blob) {
        const url = URL.createObjectURL(blob);
        const img = document.createElement('img');
        img.src = url;
        document.body.appendChild(img);

        const link = document.createElement('a');
        link.href = url;
        link.download = 'lissajous_starfields.gif';
        link.click();
    });
    gif.render();
});

// --- Go! ---
animate();

</script>
