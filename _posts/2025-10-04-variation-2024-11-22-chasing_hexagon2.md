```markdown
---
layout: fullscreen
title: "Psychedelic Waveweave"
tags:
  - graphics
---

<canvas id="waveweaveCanvas" width="600" height="600" style="display:block;margin:0 auto;"></canvas>
<script>
const canvas = document.getElementById('waveweaveCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width, H = canvas.height;
let time = 0;

// Utility: HSV to RGB
function hsv2rgb(h, s, v) {
    let f = (n, k = (n + h/60)%6) => v - v * s * Math.max(Math.min(k,4-k,1),0); 
    return [f(5)*255, f(3)*255, f(1)*255];
}

// - - - - - CONFIG - - - - - //
const bands = 6; // Number of main bands (waves)
const pointsInBand = 100; // Points per band
const freq = 2.2; // Number of wiggles per band
const layers = 18; // How many overlaid "wave layers"
const layerSpacing = 17;
const baseRadius = 115;
const colorStep = 360 / layers;
const pointJitter = 11;

function draw() {
    ctx.clearRect(0,0,W,H);
    ctx.globalCompositeOperation = 'lighter';

    for (let l = 0; l < layers; l++) {
        const t = time*0.015 + l*0.07;
        const hue = (l*colorStep + time*4)%360;
        const [r, g, b] = hsv2rgb(hue, 0.8, 0.9);
        ctx.strokeStyle = `rgba(${r|0},${g|0},${b|0}, 0.21)`;
        ctx.lineWidth = 2 + 1.1*Math.sin(t+l);

        for (let b = 0; b < bands; b++) {
            ctx.beginPath();

            for (let p = 0; p <= pointsInBand; p++) {
                let f = p / pointsInBand;
                let ang = (2*Math.PI* (b + f)/bands) - Math.PI/2;

                // Wavy band radius
                let bandRadius = baseRadius + l*layerSpacing +
                    34*Math.sin(freq*(ang + t + b*1.7)) +
                    13*Math.cos(3*ang + t*1.5 - b*2.1);

                // Evolving mid-band "centered noise"
                let n = (
                    18*Math.sin(ang*3.2 - t*1.9 + l*0.7) +
                    7*Math.sin(p*0.7 - t*2 + l)
                );

                let jitterAng = ang + (Math.sin(time*0.03+p+l)*0.07);

                const x = W/2 + (bandRadius + n)*Math.cos(jitterAng) + Math.sin(b*13+l)*pointJitter;
                const y = H/2 + (bandRadius + n)*Math.sin(jitterAng) + Math.cos(b*11+l)*pointJitter;

                if (p === 0) ctx.moveTo(x,y);
                else ctx.lineTo(x,y);
            }
            ctx.closePath();
            ctx.stroke();
        }
    }
    ctx.globalCompositeOperation = 'source-over';
}

function animate() {
    time += 1;
    draw();
    requestAnimationFrame(animate);
}

// Optional: fade-in background for a trippier effect
function fadeBg() {
    ctx.save();
    ctx.globalAlpha = 0.065;
    ctx.fillStyle = '#0a0c17';
    ctx.fillRect(0,0,W,H);
    ctx.restore();
}

function loop() {
    fadeBg();
    draw();
    time += 0.9;
    requestAnimationFrame(loop);
}

ctx.fillStyle='#0a0c17';
ctx.fillRect(0,0,W,H);
loop();

</script>
```
