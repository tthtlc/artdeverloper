---
layout: fullscreen
title: "Psychedelic Spiral Web of Interlaced Waves"
tags:
  - graphics
---

<canvas id="spiralWebCanvas" width="800" height="800"></canvas>
<script>
const canvas = document.getElementById('spiralWebCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width;
const H = canvas.height;
const cx = W/2, cy = H/2;

let t = 0;

// Utility: HSL rainbow for a smooth psychedelic palette
function rainbowColor(a, offset=0, sat=80, lum=60) {
    return `hsl(${((a * 360 + offset)%360)|0},${sat}%,${lum}%)`;
}

// Core logic: Generates a multi-armed spiral with interlaced wavy threads
function drawSpiralWeb(time) {
    ctx.clearRect(0, 0, W, H);

    const numArms = 9;
    const numRings = 22;
    const armWaveMagnitude = Math.sin(time*0.07) * 20 + 40;
    const ringWaveMag = Math.cos(time*0.123) * 6 + 16;
    const baseR = 110;
    const overallTwist = Math.sin(time*0.05) * Math.PI/8;

    // Draw wavy spiral arms
    for(let a=0; a<numArms; a++) {
        ctx.save();
        ctx.translate(cx, cy);
        const base_theta = (a * 2*Math.PI/numArms) + overallTwist;

        ctx.beginPath();
        for(let rStep=0; rStep<=numRings+3; rStep+=0.11) {
            // Spiral out:
            const r = baseR + rStep * 18 +
                Math.sin(time*0.16 + rStep*0.84 + a)*armWaveMagnitude +
                Math.cos(a*2.3+time*0.23+rStep*1.32)*ringWaveMag;

            // Local undulations for trippy effect
            const angle = base_theta +
                  rStep * 0.41
                + Math.sin(time*0.19 + rStep*1.33 + a*0.9)*0.14
                + Math.cos(time*0.057 + a*0.82 + rStep*0.42)*0.07;

            const X = Math.cos(angle)*r;
            const Y = Math.sin(angle)*r;

            if(rStep === 0) ctx.moveTo(X, Y);
            else ctx.lineTo(X, Y);
        }
        ctx.shadowColor = rainbowColor(a/numArms + time*0.015, 30,90,65);
        ctx.shadowBlur = 28;
        ctx.strokeStyle = rainbowColor(a/numArms + time*0.07, 0, 77, 52);
        ctx.lineWidth = 2.6 + 2.2*Math.sin(time*0.2+a*0.4);
        ctx.stroke();
        ctx.restore();
    }

    // Draw radial oscillating "web" threads, interlacing between arms
    for(let ring=1; ring<numRings+3; ring++) {
        ctx.save();
        ctx.translate(cx, cy);
        ctx.beginPath();

        let points = [];
        let npts = numArms*2;
        for(let i=0; i<npts+1; i++) {
            const tt = i/npts;
            const twirlFact = Math.sin(time*0.13 + ring*0.3) * 0.18 + 0.22;
            const ang = tt*2*Math.PI +
                Math.sin(ring*0.6 + time*0.069)*twirlFact +
                Math.sin(time*0.18+tt*23+ring)*0.015;

            // Wavy radius modulations for trippy effect
            const r = baseR + ring*18
                + Math.sin(ang*2 + time*0.22 + ring*0.7)*12 * Math.sin(time*0.12)
                + Math.cos(tt*4.6 + time*0.11 + ring)*8;

            const X = Math.cos(ang)*r;
            const Y = Math.sin(ang)*r;
            if(i===0) ctx.moveTo(X, Y);
            else ctx.lineTo(X, Y);
        }
        // Style alternates for the web:
        ctx.strokeStyle = rainbowColor((ring*3.1 + time*12), 30+ring*4, 95-5*(ring%2), 54-1.2*ring)+(ring%2===0? "":"");
        ctx.lineWidth = 0.7 + 0.6*Math.cos(time*0.19 + ring);
        ctx.shadowColor = rainbowColor((ring*2.1 + time*26)%360, 60, 90, 67);
        ctx.shadowBlur = 17;
        ctx.globalAlpha = 0.73;
        ctx.stroke();
        ctx.restore();
    }

    // Center oscillating void/hole
    ctx.save();
    ctx.beginPath();
    const innerR = baseR + Math.sin(time*0.08)*28;
    ctx.arc(cx, cy, Math.abs(innerR*0.64), 0, 2*Math.PI);
    ctx.globalAlpha = 0.25+0.18*Math.sin(time*0.6)+0.07*Math.cos(time);
    ctx.fillStyle = '#111014';
    ctx.shadowColor = "#fff0";
    ctx.shadowBlur = 0;
    ctx.fill();
    ctx.restore();
}

function animate() {
    t += 0.022;
    drawSpiralWeb(t);
    requestAnimationFrame(animate);
}
animate();

</script>
