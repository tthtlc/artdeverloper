---
layout: fullscreen
title: "Hypno-Vortex Spiraling Orbs"
tags:
  - graphics
---

<canvas id="vortexCanvas" width="500" height="500"></canvas>
<script>
const canvas = document.getElementById('vortexCanvas');
const ctx = canvas.getContext('2d');
const w = canvas.width;
const h = canvas.height;

let t = 0;

// Parameters for the vortex and orbs
const orbCount = 48;
const orbitRadii = Array.from({length: orbCount}, (_,i) => 50 + i*4);
const center = { x: w/2, y: h/2 };
const colors = [];
for (let i=0; i<orbCount; i++) {
    const hue = (Math.sin(i*0.17)+1)*180 + i*3;
    const sat = 60 + 30*Math.sin(i*0.37+t*0.05);
    colors.push(`hsl(${hue},${sat}%,60%)`);
}
const TAU = 2 * Math.PI;

// Animate background starfield for depth
const stars = Array.from({length:100}, () => ({
  x: Math.random()*w, y: Math.random()*h,
  r: 0.5+Math.random()*1.5, s: 0.5+Math.random()
}));

function drawStars(bgAlpha) {
    ctx.save();
    ctx.globalAlpha = bgAlpha;
    ctx.fillStyle='black';
    ctx.fillRect(0,0,w,h);
    ctx.globalAlpha = 1.0;
    for(const star of stars) {
      ctx.beginPath();
      ctx.arc(star.x,star.y,star.r,0,TAU);
      ctx.fillStyle = `rgba(255,255,${180+Math.random()*75},${0.5+Math.random()*0.5})`;
      ctx.fill();
    }
    ctx.restore();
}

// Draw vortex spiral arms
function drawVortexArms(t) {
    for(let arm=0; arm<3; arm++) {
        ctx.save();
        ctx.strokeStyle = `hsla(${210+arm*45+t*32},70%,68%,0.07)`;
        ctx.lineWidth = 4;
        ctx.beginPath();
        for(let i=0; i<180; i++) {
            const theta = (i/180)*TAU*1.2 + arm*TAU/3 + t*0.29;
            const r = 30 + i*2.2 + Math.sin(theta*3 + t*0.51)*9;
            const px = center.x + r * Math.cos(theta);
            const py = center.y + r * Math.sin(theta);
            if(i===0) ctx.moveTo(px,py);
            else ctx.lineTo(px,py);
        }
        ctx.stroke();
        ctx.restore();
    }
}

// Evolving orbs in vortex
function drawOrbs(t) {
    for(let i=0; i<orbCount; i++) {
        const baseRadius = orbitRadii[i];

        // Modulate along a spiral over time
        let theta = t*0.019 + i*TAU/orbCount*2 + Math.sin(t*0.011 + i*0.13)*0.7;
        let r = baseRadius + 32*Math.sin(t*0.009+i*0.41) + 7*Math.sin(t*0.022+i*i*0.09);

        // Position in vortex spiral
        let x = center.x + r * Math.cos(theta + t*0.003*i + Math.sin(t*0.011+ i*0.21)*0.12);
        let y = center.y + r * Math.sin(theta + Math.cos(t*0.01-i*0.15)*0.2);

        // animate orb radii, size pulses
        let orbR = 13 + 4*Math.sin(t*0.021+i*1.4) + 4*Math.sin(t*0.031+i*2.3);

        // Color shifts
        const hue = (210 + 140*Math.sin(i*0.14 + t*0.045 + Math.atan2(y-center.y,x-center.x)) + i*4 + t*0.9)%360;
        const light = 55 + 18*Math.sin(t*0.027 + i*3.9);
        const sat = 70 + 18*Math.sin(t*0.015+i*0.9);

        // Draw glowing orb
        ctx.save();
        ctx.globalAlpha = 0.19 + 0.36*(0.5 + 0.5*Math.sin(t*0.035 + i));
        ctx.beginPath();
        ctx.arc(x, y, orbR*1.4, 0, TAU);
        ctx.fillStyle = `hsla(${hue},${sat+20}%,${light+12}%,0.55)`;
        ctx.shadowColor = `hsl(${hue},${sat}%,${light+10}%)`;
        ctx.shadowBlur = 16;
        ctx.fill();
        ctx.restore();

        // Inner orb
        ctx.save();
        ctx.globalAlpha = 0.65;
        ctx.beginPath();
        ctx.arc(x, y, orbR, 0, TAU);
        ctx.fillStyle = `hsl(${hue},${sat}%,${light}%)`;
        ctx.shadowColor = `hsl(${hue-10},${Math.max(30,sat-22)}%,${Math.max(15,light-25)}%)`;
        ctx.shadowBlur = 3;
        ctx.fill();
        ctx.restore();

        // Radiant lines ("comet tails")
        ctx.save();
        for(let j=0;j<4;j++) {
            let tailTheta = theta + j*TAU/4 + t*0.03*i*j;
            ctx.beginPath();
            ctx.moveTo(x,y);
            let tx = x - orbR*2 * Math.cos(tailTheta + t*0.006*i);
            let ty = y - orbR*2 * Math.sin(tailTheta + t*0.006*i);
            ctx.lineTo(tx,ty);
            ctx.strokeStyle = `hsla(${hue},${sat}%,75%,0.22)`;
            ctx.lineWidth = 1.5;
            ctx.stroke();
        }
        ctx.restore();
    }
}

// Central vortex hole (a warped portal)
function drawCentralPortal(t) {
    ctx.save();
    const pulses = 5;
    for(let j=0;j<pulses;j++) {
        ctx.beginPath();
        let r = 30 + 11*j + 7*Math.sin(t*0.03+j*1.4 + Math.sin(t*0.01+j));
        let warp = Math.sin(t*0.012 + j*0.71)*0.11;
        for(let i=0;i<=60;i++) {
            let angle = (i/60)*TAU;
            let px = center.x + r * Math.cos(angle+warp*Math.sin(4*angle+t*0.04+j));
            let py = center.y + r * Math.sin(angle+warp*Math.cos(3*angle-t*0.024));
            if(i===0) ctx.moveTo(px,py);
            else ctx.lineTo(px,py);
        }
        ctx.closePath();
        ctx.strokeStyle = `hsla(${270+Math.sin(j+ t*0.14)*60},70%,44%,${0.12+0.07*j})`;
        ctx.lineWidth = 4-0.6*j;
        ctx.stroke();
    }
    // Portal core pulse
    ctx.beginPath();
    ctx.arc(center.x,center.y,25+2*Math.sin(t*0.045),0,TAU);
    ctx.fillStyle="radial-gradient";
    ctx.globalAlpha = 0.9;
    ctx.fillStyle = `hsl(${260+Math.sin(t*0.06)*32},88%,20%)`;
    ctx.shadowColor = `hsl(${260+Math.sin(t*0.07)*40},70%,35%)`;
    ctx.shadowBlur = 21;
    ctx.fill();
    ctx.restore();
}

// Main animation loop
function draw() {
    t += 1;
    // Background starfield fade
    drawStars(0.62+0.14*Math.sin(t*0.008));
    // Spiral arms
    drawVortexArms(t);

    // Orbs in the vortex
    drawOrbs(t);

    // Central portal
    drawCentralPortal(t);

    requestAnimationFrame(draw);
}
draw();
</script>
