---
layout: fullscreen
title: "Hypnotic Expanding Spiral Web"
tags:
  - graphics
---

<style>
canvas {
    background: radial-gradient(ellipse at center, #14014a 0%, #0b0a1e 100%);
    display: block;
    margin: 0 auto;
    border: 1px solid #222;
}
</style>
<canvas id="canvas" width="700" height="700"></canvas>
<script>
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const w = canvas.width;
const h = canvas.height;

// Utility
function lerp(a, b, t) {
    return a + (b - a) * t;
}
function hsla(h, s, l, a=1) {
    return `hsla(${h},${s}%,${l}%,${a})`;
}

const params = {
    arms: 9,
    spiralLayers: 18,
    spiralSpacing: 24,
    dotsPerLayer: 80,
    trail: 0.13 // controls afterimage
};
let t = 0;

// Main Animation
function draw() {
    // Faint afterimage for trailing effect
    ctx.globalAlpha = params.trail;
    ctx.fillStyle = "#0a0623";
    ctx.fillRect(0,0,w,h);
    ctx.globalAlpha = 1;

    ctx.save();
    ctx.translate(w/2, h/2);

    // Dynamic variables
    const baseRot = Math.sin(t*0.003)*0.1 + t*0.003; // slow rotation + sway

    // Draw spiral web
    for (let layer = 1; layer <= params.spiralLayers; layer++) {
        const r = layer * params.spiralSpacing + Math.sin(t*0.007 + layer)*18;
        const spiralPhase = t*0.0025 + layer * 0.23;
        for (let k = 0; k < params.dotsPerLayer; k++) {
            const angle = k * 2*Math.PI / params.dotsPerLayer + baseRot + spiralPhase*layer*0.05;
            // Drift makes the web wriggle slightly
            const drift = Math.sin(t*0.01 + angle*params.arms + layer)*Math.cos(t*0.008 + k*0.17);
            const rr = r + drift * 7;
            const x = Math.cos(angle) * rr;
            const y = Math.sin(angle) * rr;

            // Trails for each arm
            if (layer > 1 && k % (params.dotsPerLayer/params.arms) < 1) {
                // Connect to previous layer (radial arms)
                const prevAngle = angle - 0.011 + Math.sin(layer)*0.011;
                const px = Math.cos(prevAngle) * (rr-params.spiralSpacing);
                const py = Math.sin(prevAngle) * (rr-params.spiralSpacing);

                ctx.lineWidth = 2.5 + Math.sin(t*0.013 + layer)*0.5;
                // Colorful arms
                let hue = 220 + ((k/params.dotsPerLayer)*360 + t*0.21 + layer*13) % 360;
                let light = 60 + Math.sin(layer*0.3 + t*0.004) * 7;
                ctx.strokeStyle = hsla(hue, 75, light, 0.45 + 0.2*Math.sin(t*0.01+layer));
                ctx.beginPath();
                ctx.moveTo(x, y);
                ctx.lineTo(px, py);
                ctx.stroke();
            }

            // Spiral connections
            if (k > 0) {
                // Connect to previous dot in layer (spiral threads)
                const prevAngle2 = (k-1) * 2*Math.PI / params.dotsPerLayer + baseRot + spiralPhase*layer*0.05;
                const dx = Math.cos(prevAngle2) * rr;
                const dy = Math.sin(prevAngle2) * rr;
                ctx.lineWidth = 1.1 + Math.sin(t*0.01+layer)*0.5;
                let hue2 = (60 + (layer*38 + t*0.23 + k*2)) % 360;
                ctx.strokeStyle = hsla(hue2, 95, 76, 0.2 + 0.13*Math.sin(layer+angle+t*0.015));
                ctx.beginPath();
                ctx.moveTo(x, y);
                ctx.lineTo(dx, dy);
                ctx.stroke();
            }

            // Pulsating orb at intersection
            ctx.save();
            let orbSize = lerp(0.8,3.8, Math.abs(Math.sin(t*0.008+layer*0.41 + k*0.02))*0.92);
            ctx.beginPath();
            ctx.arc(x, y, orbSize, 0, 2*Math.PI);
            ctx.fillStyle = hsla(200+layer*7+k*1.2, 80, lerp(45,74,Math.sin(layer+k*0.14)), 0.28 + 0.08*Math.cos(t*0.01+layer));
            ctx.shadowColor = hsla(198+layer*6, 80, 56, 0.22);
            ctx.shadowBlur = 9;
            ctx.fill();
            ctx.restore();
        }
    }

    // Fractal flower at the center
    drawFractalFlower(t);

    ctx.restore();
    t += 1;
    requestAnimationFrame(draw);
}

function drawFractalFlower(tt) {
    const petals = 17;
    const R = 34 + Math.sin(tt*0.013)*6;
    ctx.save();
    ctx.globalAlpha = 0.87;
    for (let p = 0; p < petals; p++) {
        const phi = (p/petals)*Math.PI*2 + Math.cos(tt*0.0017)*0.5;
        ctx.save();
        ctx.rotate(phi + Math.sin(tt*0.002 + p)*0.1);
        // Petal color
        let baseHue = ((tt*0.14)%360 + p*24) % 360;
        ctx.fillStyle = hsla(baseHue, 80, 74, 0.38 + 0.2*Math.sin(tt*0.015+p));
        ctx.beginPath();
        ctx.moveTo(0,0);
        ctx.bezierCurveTo(R*0.2,-R*0.46, R,R*0.4, 0, R*1.35 + Math.sin(tt*0.03+p)*3.5);
        ctx.bezierCurveTo(-R,R*0.4, -R*0.2,-R*0.46, 0,0);
        ctx.closePath();
        ctx.shadowColor = hsla((baseHue+45)%360, 85, 58, 0.24);
        ctx.shadowBlur = 9;
        ctx.fill();
        ctx.restore();
    }
    // Core
    ctx.beginPath();
    ctx.arc(0,0,lerp(16,28,Math.abs(Math.sin(tt*0.016))),0,2*Math.PI);
    ctx.fillStyle = hsla(70+Math.sin(tt*0.022)*67,96,67,0.77);
    ctx.shadowColor='rgba(255,255,215,0.32)';
    ctx.shadowBlur=22;
    ctx.fill();
    ctx.restore();
}

draw();
</script>