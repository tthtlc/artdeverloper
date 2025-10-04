---
layout: fullscreen
title: "Psychedelic Moiré Flower"
tags:
  - graphics
---

<canvas id="moireFlower" width="600" height="600" style="background:#111;display:block;margin:0 auto;"></canvas>
<script>
const canvas = document.getElementById('moireFlower');
const ctx = canvas.getContext('2d');

const w = canvas.width, h = canvas.height;
const cx = w/2, cy = h/2;
let t = 0;

function hsv(h, s, v) {
    h = h%360;
    s = Math.min(1,Math.max(0,s));
    v = Math.min(1,Math.max(0,v));
    let C = v*s, X = C*(1-Math.abs((h/60)%2-1)), m = v-C;
    let [r,g,b]=[0,0,0];
    if (h < 60) [r,g,b]=[C,X,0];
    else if (h < 120) [r,g,b]=[X,C,0];
    else if (h < 180) [r,g,b]=[0,C,X];
    else if (h < 240) [r,g,b]=[0,X,C];
    else if (h < 300) [r,g,b]=[X,0,C];
    else [r,g,b]=[C,0,X];
    return `rgb(${Math.round(255*(r+m))},${Math.round(255*(g+m))},${Math.round(255*(b+m))})`;
}

function drawPetal(r1, r2, a1, a2, phase, k, color) {
    // Draws a single petal using curved lines in moiré-lattice
    ctx.save();
    ctx.strokeStyle = color;
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    for (let t=0; t<=1.0001; t+=0.02) {
        // Lerp between two arcs, but with wave-warped radius
        let angle = a1*(1-t) + a2*t;
        let tr = r1*(1-t) + r2*t;
        let warp = 20 * Math.sin(phase + 6 * angle + k * t);
        let x = cx + (tr+warp)*Math.cos(angle);
        let y = cy + (tr+warp)*Math.sin(angle);
        if(t===0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
    }
    ctx.stroke();
    ctx.restore();
}

function drawMoiréFlower(time) {
    ctx.clearRect(0,0,w,h);
    let petals = 16;
    let layers = 25;
    let baseR = 80 + 60 * Math.sin(time/900);
    let kMotion = Math.sin(time/2500)*0.8 + 1.2;
    for(let p = 0; p < petals; ++p) {
        let baseAngle = (2*Math.PI/petals)*p + Math.sin(time/1100)*0.23;
        for(let l = 0; l < layers; ++l) {
            let tLayer = l/(layers-1);
            let radius1 = baseR + tLayer*190 + 18.5*Math.sin(time/400 + l*0.2 + p*0.1);
            let radius2 = baseR + (tLayer+0.035)*190 + 16*Math.cos(time/690 + l*0.32);
            let angle1 = baseAngle + 0.18 * Math.sin(time/1200 + l*0.13);
            let angle2 = baseAngle + 2*Math.PI/petals + 0.18 * Math.cos(time/1200 + l*0.11);

            let hue = (p*360/petals + l*7 + time/22) % 360;
            let bright = 0.51 + 0.41*Math.sin(time/950 + l*0.7 + p*0.13);
            let color = hsv(hue, 0.82-Math.abs(0.5-tLayer)*0.7, bright);

            drawPetal(radius1, radius2, angle1, angle2, time/300 + l*0.22 + p*0.11, kMotion, color);
        }
    }
    // Center swirl
    ctx.save();
    ctx.globalAlpha=0.26;
    for(let i=0;i<8;++i) {
        let ang = i*(Math.PI/4)+Math.cos(time/1500)*0.6;
        let R = baseR*0.61 + 19*Math.sin(time/260 + i*0.9);
        let x = cx + R*Math.cos(ang);
        let y = cy + R*Math.sin(ang);
        ctx.beginPath();
        ctx.arc(x,y,34+3*Math.sin(time/900+i),0,2*Math.PI);
        ctx.strokeStyle=hsv(i*45+time/4,0.62,1);
        ctx.lineWidth = 7 + 5*Math.sin(time/1900+i*0.4);
        ctx.stroke();
    }
    ctx.restore();
}

function animate() {
    t = performance.now();
    drawMoiréFlower(t);
    requestAnimationFrame(animate);
}

animate();
</script>
