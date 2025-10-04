---
layout: fullscreen
title: "Oscillating Kaleidoscopic Star Web"
tags:
  - graphics
---

<canvas id="psyStarWeb" width="720" height="720"></canvas>
<script>
const canvas = document.getElementById('psyStarWeb');
const ctx = canvas.getContext('2d');
const w = canvas.width, h = canvas.height;
const cx = w/2, cy = h/2;

// Parameters for the generative star field
const STAR_POINTS = 11;
const BASE_RADIUS = 140;
const INNER_RADIUS = 52;
const WAVE_DEPTH = 50; // Amount of radial modulation
const PARTICLE_COUNT = 120;
const PARTICLE_ARCS = 5;
const LINES_PER_FRAME = 65;
const COLOR_FREQ = 0.9;
const SAT_BASE = 90;
const LIGHT_BASE = 60;

let time = 0;

// Helper: get star-shaped points
function starPoints(cx, cy, arms, r1, r2, t, phase=0) {
    const pts = [];
    for (let i=0; i<arms; i++) {
        const theta = (i / arms) * 2 * Math.PI + phase;
        // Animate radii as waves
        let angleWave = Math.sin(theta * 3 + t * 1.2) * WAVE_DEPTH;
        let rA = r1 + angleWave * 0.6 * Math.sin(t + i * 0.13);
        let rB = r2 + angleWave * 0.2 * Math.cos(t*1.4 + i * 0.23);
        pts.push([
            cx + Math.cos(theta) * rA,
            cy + Math.sin(theta) * rA
        ]);
        pts.push([
            cx + Math.cos(theta + Math.PI/arms) * rB,
            cy + Math.sin(theta + Math.PI/arms) * rB
        ]);
    }
    return pts;
}

// Dynamic particle system cycling in rings
let particles = [];
function resetParticles(t) {
    particles.length = 0;
    for (let i=0; i<PARTICLE_COUNT; i++) {
        // Scatter in spiral
        const baseTheta = (i/PARTICLE_COUNT) * 2 * Math.PI * PARTICLE_ARCS + Math.sin(t+i)*0.21;
        const r = BASE_RADIUS * 0.5 + Math.sin(t*1.1 + i)*47 + (i*2)%100;
        const theta = baseTheta + Math.cos(i)*0.04 + Math.sin(t + i*2)*0.33;
        let px = cx + Math.cos(theta) * r;
        let py = cy + Math.sin(theta) * r;
        // Animate movement along angle
        px += Math.cos(theta + t*0.9) * 24 * Math.cos(t+i*0.177);
        py += Math.sin(theta - t*0.6) * 19 * Math.sin(t+i*0.057);
        particles.push({x: px, y: py, a: theta, idx: i});
    }
}

// Color helpers
function hsl(h, s, l, a=1) {
    return `hsla(${h},${s}%,${l}%,${a})`;
}

// Draw thin radial web lines for stars & between particles
function drawStarWeb(t, pts, layers = 3) {
    for (let d=0; d<layers; d++) {
        ctx.save();
        ctx.globalAlpha = 0.15 + 0.10*Math.sin(t + d*1.1);
        ctx.strokeStyle = hsl(190 + 90*Math.sin(t + d*0.7), SAT_BASE+6*d, 62+8*Math.cos(t*0.25+d), 0.6);
        ctx.lineWidth = 1 + 0.33*Math.abs(Math.sin(t+d));
        ctx.beginPath();
        for (let i=0; i<pts.length; i++) {
            const [x1, y1] = pts[i];
            const [x2, y2] = pts[(i+1)%pts.length];
            ctx.moveTo(x1, y1);
            ctx.lineTo(x2, y2);
        }
        ctx.stroke();
        ctx.restore();
    }
}

// Draw wave-animated polygons with rainbow glows
function drawTrippyStars(t, layers=3) {
    for (let jj=layers; jj>0; jj--) {
        // Animate phase
        let phase = t*0.3 + jj * 0.17;
        let rA = BASE_RADIUS + (jj-1)*22 + Math.sin(t + jj)*11;
        let rB = INNER_RADIUS + (jj-1)*15 + Math.cos(t*0.75 + jj)*5;
        const pts = starPoints(cx, cy, STAR_POINTS, rA, rB, t, phase);
        
        ctx.save();
        ctx.shadowColor = hsl(70 + Math.abs(Math.sin(t+jj))*250, 95, 55, 0.25);
        ctx.shadowBlur = 32 + 24*jj;
        ctx.globalAlpha = 0.68 - 0.15*jj;
        let hue = 240 + 75*Math.sin(t*0.5 + jj*1.3);
        ctx.strokeStyle = hsl(hue, SAT_BASE + jj*4, LIGHT_BASE + 6*jj, 0.85);
        ctx.lineWidth = 2.4 + 0.6*jj;
        ctx.beginPath();
        for (let i=0; i<pts.length; i++) {
            const [x, y] = pts[i];
            if (i===0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        ctx.closePath();
        ctx.stroke();
        ctx.restore();

        drawStarWeb(t + jj*0.09, pts, 2+(jj%2));
    }
}

// Draw dynamic “web” lines between star vertices & particles
function drawWebs(t, pts, particles) {
    ctx.save();
    ctx.globalAlpha = 0.15 + 0.05*Math.sin(t*1.6);
    ctx.lineWidth = 0.5;
    for (let i=0; i<particles.length; i += 2) {
        const {x, y, a, idx} = particles[i];
        let n = (i * 5) % pts.length;
        let target = pts[n];
        let hue = 60 + 220*(i/PARTICLE_COUNT) + Math.sin(t + i)*60;
        ctx.strokeStyle = hsl(hue, 80, 53, 0.65-0.27*Math.sin(idx+t*0.7));
        ctx.beginPath();
        ctx.moveTo(x, y);
        ctx.lineTo(target[0], target[1]);
        ctx.stroke();
    }
    ctx.restore();
}

// Animate glowing lines orbiting between particles
function drawParticleArcs(t, particles, count=LINES_PER_FRAME) {
    ctx.save();
    for (let i=0; i<count; i++) {
        const p1 = particles[(i*41)%particles.length];
        const p2 = particles[(i*37 + 13)%particles.length];
        // Animate color along angle
        let hue = 120 + 150*Math.sin((p1.idx + t*1.1)*COLOR_FREQ);
        ctx.globalAlpha = 0.19 + 0.10*Math.sin(t + i*0.12);
        ctx.lineWidth = 1.07 + 0.14*Math.cos(i + t*0.71);
        ctx.strokeStyle = hsl(hue, SAT_BASE, 44+Math.sin(t+i)*15, 0.6);
        ctx.beginPath();
        ctx.moveTo(p1.x, p1.y);
        ctx.lineTo(p2.x, p2.y);
        ctx.stroke();
    }
    ctx.restore();
}

// Main render loop
function animate() {
    ctx.clearRect(0,0,canvas.width,canvas.height);

    let bgCol = 250 + 45*Math.sin(time*0.19);
    ctx.fillStyle = hsl(bgCol, 35, 8, 1);
    ctx.fillRect(0,0,canvas.width,canvas.height);

    // Animate stars
    drawTrippyStars(time, 4);

    // Animate particle ring & webs
    resetParticles(time);
    const corePts = starPoints(cx, cy, STAR_POINTS, BASE_RADIUS, INNER_RADIUS, time, time*0.19);
    drawWebs(time, corePts, particles);
    drawParticleArcs(time, particles, LINES_PER_FRAME);

    time += 0.0125;
    requestAnimationFrame(animate);
}
animate();
</script>
