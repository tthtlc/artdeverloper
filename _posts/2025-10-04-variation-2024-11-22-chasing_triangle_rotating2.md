---
layout: fullscreen
title: "Hypnotic Sinusoidal Web"
tags:
  - graphics
---

<canvas id="webCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('webCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width;
const H = canvas.height;

// Vivid rainbow-style palette for sinuous cycling
function hsv2rgb(h, s, v) {
    let f = (n, k = (n + h/60) % 6) =>
        v - v*s * Math.max(Math.min(k,4-k,1),0);
    return `rgb(${f(5)*255},${f(3)*255},${f(1)*255})`;
}

// Parameters
const webArms = 17; // Number of radiant web lines (choose prime for more psychedelia)
const maxRings = 42;
const webLayers = 7; // How many overlaid web fragments per frame
const pointsPerArm = 120; // Points along each line
const baseAmplitude = 90;

let t = 0;

function drawWeb() {
    ctx.clearRect(0, 0, W, H);

    for (let layer = 0; layer < webLayers; layer++) {
        let timeOffset = t*0.013 + Math.sin(t/101 + layer) * 2.1;
        let phase = Math.PI*2 * layer / webLayers;

        ctx.save();
        ctx.globalAlpha = 0.40 + 0.25*Math.sin(t/80 + layer);
        ctx.lineWidth = 1 + 1*Math.abs(Math.sin(phase + t*0.01));
        
        // Each arm
        for (let k = 0; k < webArms; k++) {
            let armAngle = (Math.PI*2/webArms)*k + phase;
            let hue = (360*(k/webArms) + 70*t/60 + layer*17)%360;
            let sat = 0.82 + 0.18*Math.sin(layer + k + t/200);
            let val = 0.88 + 0.1*Math.cos(k - t/400 + layer);

            ctx.strokeStyle = hsv2rgb(hue, sat, val);

            ctx.beginPath();
            for (let p = 0; p < pointsPerArm; p++) {
                let pr = p / (pointsPerArm-1);

                // Sinusoidal radius modulation
                let sinFreq = 3.6 + 0.9*Math.sin(layer + t/180 + k/6);
                let wave = Math.sin(
                    phase + pr*2.5 + sinFreq*pr*5 + armAngle + timeOffset*0.6
                    ) * (0.6 + 0.5*Math.sin(timeOffset + layer*0.4));

                let radius = pr*(H/2.1) + 
                    (baseAmplitude + 16*layer) * wave * (1 - pr*0.19);

                let x = W/2 + radius * Math.cos(armAngle);
                let y = H/2 + radius * Math.sin(armAngle);

                // Add extra cartesian offset for shifting
                x += (Math.cos(phase+layer+t/120) * (10+7*layer)) * Math.sin(pr*4);
                y += (Math.sin(phase+layer*0.9-t/137) * (9+10*layer)) * Math.cos(pr*5+armAngle);

                if (p === 0) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            }
            ctx.stroke();
        }
        ctx.restore();
    }

    // Draw hypnotic ripples at center
    for (let ring = 0; ring < maxRings; ring++) {
        let rad = ring*5 + 17 + (Math.sin(t/60 + ring*0.09) * 10 + 20 * Math.abs(Math.cos(t/300 + ring/9)));
        ctx.beginPath();
        ctx.arc(W/2, H/2, rad, 0, 2*Math.PI);
        let hue = (t/2 + ring*13)%360;
        ctx.strokeStyle = hsv2rgb(hue, 0.27 + 0.55*Math.abs(Math.sin(ring/8)), 0.4+0.6*Math.abs(Math.cos(ring/8)));
        ctx.globalAlpha = 0.12 + 0.17 * Math.abs(Math.sin(ring+t/420));
        ctx.lineWidth = 2.2 + 2*Math.cos(t/100+ring/6);
        ctx.stroke();
    }
    ctx.globalAlpha = 1.0;
}

function animate() {
    t++;
    drawWeb();
    requestAnimationFrame(animate);
}

animate();

</script>