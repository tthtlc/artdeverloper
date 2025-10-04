```markdown
---
layout: fullscreen
title: "Psychedelic Hypno-Waves on Parametric Rings"
tags:
  - graphics
---

This hypnotic generative art features several undulating, color-shifting rings whose waveforms, connections, and colors morph over time. Twisting parametric functions and trigonometric modulations create a mesmerizing psychedelic effect. Play with the controls to change ring count, wave complexity, and color cycling!

<canvas id="hypnoCanvas" width="800" height="800"></canvas>
<div class="controls">
    <div class="control-container">
        <label for="numRings">Number of Rings:</label>
        <input type="range" id="numRings" min="2" max="7" value="4">
        <span id="numRingsVal">4</span>
    </div>
    <div class="control-container">
        <label for="ptsPerRing">Points per Ring:</label>
        <input type="range" id="ptsPerRing" min="20" max="400" value="180">
        <span id="ptsPerRingVal">180</span>
    </div>
    <div class="control-container">
        <label for="waveComplexity">Wave Complexity:</label>
        <input type="range" id="waveComplexity" min="1" max="12" value="5">
        <span id="waveComplexityVal">5</span>
    </div>
    <div class="control-container">
        <label for="colorSpeed">Color Cycle Speed:</label>
        <input type="range" id="colorSpeed" min="1" max="20" value="6">
        <span id="colorSpeedVal">6</span>
    </div>
    <div class="control-container">
        <label for="connectionWaves">Connection Wave Offset:</label>
        <input type="range" id="connectionWaves" min="0" max="24" value="7">
        <span id="connectionWavesVal">7</span>
    </div>
</div>

<script>
const canvas = document.getElementById('hypnoCanvas');
const ctx = canvas.getContext('2d');

let w = canvas.width;
let h = canvas.height;
let cx = w/2;
let cy = h/2;

function $(id) { return document.getElementById(id); }

const numRingsSlider = $('numRings');
const ringPtsSlider = $('ptsPerRing');
const waveSlider = $('waveComplexity');
const colSpeedSlider = $('colorSpeed');
const connWaveSlider = $('connectionWaves');

const numRingsVal = $('numRingsVal');
const ringPtsVal = $('ptsPerRingVal');
const waveVal = $('waveComplexityVal');
const colSpeedVal = $('colorSpeedVal');
const connWaveVal = $('connectionWavesVal');

// Update label on input
function updateLabels() {
    numRingsVal.textContent = numRingsSlider.value;
    ringPtsVal.textContent = ringPtsSlider.value;
    waveVal.textContent = waveSlider.value;
    colSpeedVal.textContent = colSpeedSlider.value;
    connWaveVal.textContent = connWaveSlider.value;
}
[numRingsSlider, ringPtsSlider, waveSlider, colSpeedSlider, connWaveSlider].forEach(sl => {
    sl.addEventListener('input', updateLabels);
});
updateLabels();

function hsl(a, s, l) {
    // Convenience
    return `hsl(${a},${s}%,${l}%)`
}

function getParametricRingPoints(ringIndex, time, params) {
    // Params: {numPts, baseR, waveLayers, waveComplexity, seed, phaseSpeed, amplitude}
    // Returns array of {x, y}
    let pts = [];
    let {
        numPts, baseR, waveLayers, waveComplexity, seed, phaseSpeed, amplitude
    } = params;
    for (let i = 0; i < numPts; i++) {
        let ang = (i/numPts)*2*Math.PI;
        // Several sine waves of different frequency & phase add to make the final radial offset
        let r =
            baseR +
            Math.sin(ang*waveComplexity + seed + time*phaseSpeed) * amplitude/1.3 +
            Math.sin(ang*(waveComplexity+2) - seed*0.7 + time*phaseSpeed*0.7) * amplitude/2.5 +
            Math.sin(ang*(waveComplexity*1.5) + seed*2 - time*phaseSpeed*1.2) * amplitude/3.5;

        let x = cx + Math.cos(ang) * r;
        let y = cy + Math.sin(ang) * r;
        pts.push({x, y, ang});
    }
    return pts;
}

function draw() {
    ctx.setTransform(1,0,0,1,0,0);
    ctx.clearRect(0, 0, w, h);

    // Read controls
    const numRings = parseInt(numRingsSlider.value);
    const ptsPerRing = parseInt(ringPtsSlider.value);
    const waveComplexity = parseInt(waveSlider.value);
    const colorCycling = parseFloat(colSpeedSlider.value) * 0.007;
    const connectionWaves = parseInt(connWaveSlider.value);

    const t = performance.now() * 0.001;

    // For each ring, get its parameters and point set
    const rings = [];
    for (let r = 0; r < numRings; r++) {
        // Space radius for rings
        let frac = r / (numRings-1);
        let baseR = 130 + frac * 270;
        // Each ring's own wave params based on ring index & time
        let params = {
            numPts: ptsPerRing,
            baseR: baseR + Math.sin(t*0.5 + r*2.24) * 40 + Math.cos(r*1.8) * 18,
            waveLayers: 3,
            waveComplexity: waveComplexity + Math.sin(r + t*0.17)*1.9,
            seed: r * 16.9181 + Math.sin(t*0.4)*12,
            phaseSpeed: 0.37 + 0.11*r + Math.sin(t*0.25+r)*0.11,
            amplitude: 34 + Math.cos(t*0.8+ r*1.66)*14 + r*4
        };
        let pts = getParametricRingPoints(r, t, params);
        rings.push(pts);
    }

    // === Draw connections between rings ===
    for (let r = 0; r < numRings-1; r++) {
        const ptsA = rings[r];
        const ptsB = rings[r+1];
        for (let i = 0; i < ptsPerRing; i++) {
            // Connect from ring r to r+1
            // Offset the index with a wave to create dynamic twisting structure
            let iB = Math.floor(
                (i +
                Math.round(
                    Math.sin(i/ptsPerRing * connectionWaves*2*Math.PI + t*1.07 + r)
                    * (ptsPerRing/numRings)*0.48
                )) % ptsPerRing
            );
            // Color is based on global index, ring, and time
            let hue = ((i*2.9 + r*18 + t*90*colorCycling)%360);
            let sat = 60 + 30*Math.sin(t*0.4 + r);
            let lum = 36 + Math.sin(r*1.3 + i/ptsPerRing*2*Math.PI + t*0.75)*11;
            ctx.beginPath();
            ctx.moveTo(ptsA[i].x, ptsA[i].y);
            ctx.lineTo(ptsB[iB].x, ptsB[iB].y);
            ctx.strokeStyle = hsl(hue, sat, lum);
            ctx.lineWidth = 1.2 + 1.1*Math.cos(i/ptsPerRing * Math.PI + t*0.49 + r);
            ctx.globalAlpha = 0.54 + 0.35*Math.sin(i/ptsPerRing*4.0 + t*0.66 + r);
            ctx.stroke();
        }
    }

    ctx.globalAlpha = 1.0;

    // === Draw the parametric rings themselves ===
    for (let r = 0; r < numRings; r++) {
        ctx.save();
        ctx.beginPath();
        let pts = rings[r];
        ctx.moveTo(pts[0].x, pts[0].y);
        for (let i=1; i<pts.length; i++) ctx.lineTo(pts[i].x, pts[i].y);
        ctx.closePath();
        let hue = (t*90*colorCycling + r*46)%360;
        let sat = 70 + Math.cos(t*0.6 + r*0.9)*20;
        let lum = 22 + 12*r + Math.sin(t*0.57 + r*1.1)*9;
        ctx.strokeStyle = hsl(hue, sat, lum);
        ctx.lineWidth = 2.3 + 0.85*Math.sin(t*0.81+r);
        ctx.shadowColor = hsl((hue+60)%360,90,55);
        ctx.shadowBlur = 26 + 12*Math.sin(t*0.7+r*0.9);
        ctx.stroke();
        ctx.restore();
    }

    // Optionally, paint a faint glowing center
    ctx.save();
    let radgrad = ctx.createRadialGradient(cx, cy, 12, cx, cy, 172);
    radgrad.addColorStop(0, hsl((t*90*colorCycling)%360, 95, 50));
    radgrad.addColorStop(0.4, "rgba(0,0,0,0)");
    radgrad.addColorStop(1, "rgba(0,0,0,0)");
    ctx.globalAlpha = 0.12;
    ctx.beginPath();
    ctx.arc(cx, cy, 200, 0, 2*Math.PI);
    ctx.fillStyle = radgrad;
    ctx.fill();
    ctx.restore();

    requestAnimationFrame(draw);
}

draw();

</script>
```