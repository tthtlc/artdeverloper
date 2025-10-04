```markdown
---
layout: fullscreen
title: "Psychedelic Spiral Waveform Blossoms"
tags:
  - graphics
---

A hypnotic dance of ever-blooming spiral blossoms, made of colorful wave-warped petals that flicker and morph in a vibrant, trippy flow.

<canvas id="blossomCanvas" width="800" height="800"></canvas>
<script>
const canvas = document.getElementById('blossomCanvas');
const ctx = canvas.getContext('2d');
canvas.width = canvas.height = 800;
const W = canvas.width;
const H = canvas.height;
let t = 0;

/**
 * Returns a psychedelic color given an angle and a time.
 */
function rainbowColor(angle, time, alpha = 1) {
    // Hue shift for vibrant gradient
    let hue = ((angle * 180 / Math.PI) + time * 80) % 360;
    let sat = 75 + 25 * Math.sin(time + angle * 3);
    let light = 50 + 10 * Math.cos(angle * 2 - time * 1.2);
    return `hsla(${hue},${sat}%,${light}%,${alpha})`;
}

/**
 * Draw a morphing psychedelic spiral blossom with petals made from sine-warped curves.
 */
function drawBlossom(cx, cy, baseRadius, nPetals, globalPhase, layer, t) {
    ctx.save();
    ctx.translate(cx, cy);
    const steps = 200; // points per petal
    const petalPhase = globalPhase + layer * 2.2;
    const timeWobble = Math.sin(t * 0.85 + layer * 1.4) * 0.7;

    for (let p = 0; p < nPetals; p++) {
        ctx.beginPath();
        for (let i = 0; i <= steps; i++) {
            // Angle along petal
            let ang = Math.PI * i / steps;
            // Radial ripple per layer/petal
            let ripple =
                Math.sin(ang * 2 + t * 1.6 + p * 0.7 + layer * 0.5) * 0.25 +
                Math.cos(ang * 5 - t * 2.2 + layer * 1.1) * 0.12;
            // Dynamic petal length
            let R =
                baseRadius * (0.76 + 0.34 * Math.sin(t * 0.9 + layer * 2))
                + baseRadius * 0.25 * Math.sin(ang * 2 + timeWobble + t * 1.2);
            // Boundary of petal
            let r = R * (0.45 + 0.5 * Math.sin(ang + Math.cos(t * 0.5) * 0.3))
                * (1 + ripple * 0.4 + Math.sin(globalPhase + ang * 3 + t * 0.5) * 0.13);

            // Point around blossom
            let petalAng = ((2 * Math.PI) / nPetals) * p;
            let theta = petalAng + ang - Math.PI / 2
                + Math.sin(layer * 0.3 + t * 0.25) * 0.06;

            let x = Math.cos(theta) * r;
            let y = Math.sin(theta) * r;

            if (i === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        ctx.closePath();
        let color = rainbowColor(petalPhase + p * 2.31, t + layer * 1.1, 0.6 + 0.3 * Math.sin(t + p));
        ctx.fillStyle = color;
        ctx.shadowColor = color;
        ctx.shadowBlur = 40 + 22 * Math.sin(t + p*0.43 + layer*0.8);
        ctx.globalAlpha = 0.85 + 0.12 * Math.sin(t * 1.7 + p + layer);
        ctx.fill();
    }
    ctx.restore();
}

function drawBackground(t) {
    // Subtle, shifting vignette
    let grad = ctx.createRadialGradient(W/2, H/2, 40 + 20*Math.sin(t*0.3), W/2, H/2, W/1.04);
    grad.addColorStop(0, "#232836");
    grad.addColorStop(1, `hsl(${170 + 40*Math.sin(t*0.2)},60%,10%)`);
    ctx.fillStyle = grad;
    ctx.fillRect(0,0,W,H);
}

// Master animation loop
function draw() {
    t += 0.016;
    drawBackground(t);

    // Blossom system in a spiral
    const nLayers = 8;
    const nPetals = 7 + Math.floor(2 * Math.sin(t*0.2));
    let spiralR = 190 + Math.sin(t*0.3)*25;

    for (let layer = nLayers-1; layer >= 0; layer--) {
        // Spiral arrangement and evolution over time
        let spiralAngle = t*0.20 + layer * 0.73 + Math.sin(t*0.8+layer)*0.3;
        let cx = W/2 + Math.cos(spiralAngle) * spiralR * (1 - layer * 0.11);
        let cy = H/2 + Math.sin(spiralAngle) * spiralR * (1 - layer * 0.11);

        let baseRadius = 95 - layer * 6.5 + Math.sin(t*0.416 + layer*2.7) * 18;

        let globalPhase = t * (0.45 + 0.04*layer) + Math.sin(layer*1.3 + t*0.417) * 0.5;

        drawBlossom(cx, cy, baseRadius, nPetals + ((layer%2)?1:0), globalPhase, layer, t);
    }

    // Center blossom, largest, on top
    drawBlossom(W/2, H/2, 105 + Math.sin(t*0.95)*12, 10 + Math.floor(Math.sin(t*0.5)*3), t*0.33, 0, t);
    requestAnimationFrame(draw);
}

draw();
</script>
```