---
layout: fullscreen
title: "Fractal Sunburst: Blooming Rhythmic Circles"
tags:
  - graphics
---

<canvas id="sunburstCanvas" width="700" height="700" style="background:black;display:block;margin:0 auto;"></canvas>
<script>
const canvas = document.getElementById('sunburstCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
const cx = W/2, cy = H/2;

// ---- Parameters ----
let t = 0;
const petalLayers = 6;        // How many circular layers of petals
const petalsPerLayer = 24;    // Petals around each layer
const baseRadius = 60;        // Starting radius of innermost layer
const layerGap = 38;          // Distance between petal rings
const trailAlpha = 0.10;      // 0.0 = no trail, 1.0 = no fade-out

// ---- Utilities ----
function pol2cart(r, theta) {
    return [cx + r * Math.cos(theta), cy + r * Math.sin(theta)];
}

// ---- Draw a single petal (ellipse) ----
function drawPetal(r, theta, scale, color, wobble=0) {
    ctx.save();
    const [x, y] = pol2cart(r, theta);
    ctx.translate(x, y);
    ctx.rotate(theta + Math.PI/2 + wobble); // orient petal outward

    ctx.beginPath();
    ctx.ellipse(0, 0, 18 * scale, 44 * scale, 0, 0, 2*Math.PI);
    ctx.fillStyle = color;
    ctx.shadowColor = 'white';
    ctx.shadowBlur = 22*scale;
    ctx.globalAlpha = 0.96;
    ctx.fill();
    ctx.globalAlpha = 1.0;
    ctx.restore();
}

// ---- Main Animation ----
function drawSunburst(t) {
    // Fading trails effect
    ctx.globalAlpha = 1-trailAlpha;
    ctx.fillStyle = "#000";
    ctx.fillRect(0,0,W,H);
    ctx.globalAlpha = 1.0;

    // Central radiating waves
    for (let k=0; k<4; k++) {
        let waveR = baseRadius + Math.sin(t/55 + k)*layerGap*3 + k*layerGap*1.1;
        let waveColor = `hsl(${(t*0.5 + k*66)%360}, 96%, 48%)`;
        ctx.beginPath();
        ctx.arc(cx, cy, waveR, 0, 2*Math.PI);
        ctx.strokeStyle = waveColor;
        ctx.globalAlpha = 0.25 + 0.08*Math.cos(t/19+k);
        ctx.lineWidth = 8 + 5*Math.sin(t/33+k*0.5);
        ctx.shadowColor = waveColor;
        ctx.shadowBlur = 16 + Math.sin(t/30+k)*4;
        ctx.stroke();
        ctx.globalAlpha = 1.0;
    }
    ctx.shadowBlur = 0;

    // Draw petal layers (fractal, rotating, pulsating)
    for (let l=0; l<petalLayers; l++) {
        let R = baseRadius + l*layerGap + 12*Math.sin(t/64+l);
        let petalN = petalsPerLayer + Math.floor(6 * Math.sin(t/140 + l));
        let rot = t/53 + l*Math.PI/8 + Math.sin(t/103+l*18); // per-layer rotation
        for (let j=0; j<petalN; j++) {
            let phi = rot + 2*Math.PI * j / petalN;
            // Each petal's "breathe" and wobble
            let sc = 0.75 + 0.20*Math.sin(t/36+l*2+phi*2+t/73)
                         + 0.11*Math.sin(t/17+phi*6+l*9+t/103);
            let wobble = 0.15 * Math.sin(t/54 + phi*3 + l*6) + 
                         0.07*Math.cos(t/75+phi*8+t/130);
            // Trippy color per petal and layer
            let hue = (t*1.6 + (360/petalLayers)*l + 220 + phi*85) % 360;
            let sat = 85 + 15*Math.sin(t/39-phi*2);
            let lum = 61 + 18*Math.cos(t/43+phi*2+l*13);
            drawPetal(R, phi, sc, `hsl(${hue},${sat}%,${lum}%)`,wobble);
        }
    }

    // Central pulsing core
    let corePulse = 24 + 22*Math.abs(Math.sin(t/47));
    let coreColor = `hsl(${(t*2.2)%360},98%,65%)`;
    ctx.beginPath();
    ctx.arc(cx, cy, corePulse, 0, 2*Math.PI);
    ctx.globalAlpha = 0.78+0.15*Math.sin(t/111);
    ctx.fillStyle = coreColor;
    ctx.shadowColor = "#fff";
    ctx.shadowBlur = 48 + 20*Math.sin(t/23);
    ctx.fill();
    ctx.globalAlpha = 1.0;
    ctx.shadowBlur = 0;

    // Central starburst tips with fractal recursion
    function drawStar(cx, cy, rad, arms, thick, depth, rot=0) {
        if (depth<=0) return;
        for (let a=0; a<arms; a++) {
            let ang = rot + a*2*Math.PI/arms;
            let x = cx + rad * Math.cos(ang);
            let y = cy + rad * Math.sin(ang);
            ctx.save();
            ctx.strokeStyle = `hsl(${(t*4+a*38)%360},100%,85%)`;
            ctx.lineWidth = thick * (0.9+0.20*Math.sin(t/56+ang*5));
            ctx.beginPath();
            ctx.moveTo(cx,cy);
            ctx.lineTo(x,y);
            ctx.globalAlpha = 0.33 + 0.15*Math.sin(t/61+a*7);
            ctx.shadowColor = '#fff';
            ctx.shadowBlur = 18;
            ctx.stroke();
            ctx.globalAlpha = 1.0;
            ctx.shadowBlur = 0;
            ctx.restore();

            // Recursive arms
            if (depth>1) {
                drawStar(x, y, rad*0.43, arms, thick*0.47, depth-1, ang+t/90);
            }
        }
    }
    drawStar(cx,cy,corePulse+19,8,7,3,t/210);

    ctx.shadowBlur = 0;
} // drawSunburst

function animate() {
    t += 1;
    drawSunburst(t);
    requestAnimationFrame(animate);
}
animate();
</script>
