```markdown
---
layout: fullscreen
title: "Psychedelic Spiro-Particle Vortex"
tags:
  - graphics
---

<canvas id="spiro-vortex" style="display:block; width:100vw; height:100vh; position:fixed; top:0; left:0; z-index:-1; background:#110421;"></canvas>

<script>
const canvas = document.getElementById('spiro-vortex');
const ctx = canvas.getContext('2d');

let w = window.innerWidth;
let h = window.innerHeight;
canvas.width = w;
canvas.height = h;

function resize() {
    w = window.innerWidth;
    h = window.innerHeight;
    canvas.width = w;
    canvas.height = h;
}
window.addEventListener('resize', resize);

// --- Spiro-Particle System ---

const N_SPIRO = 8;
const SPIRO_PARTICLES = 70;
let spiros = [];

function randomColor() {
    // Colorful neon using HSL
    return `hsl(${Math.floor(Math.random()*360)}, 92%, 62%)`;
}

class Spiro {
    constructor(cx, cy, r1, r2, speed, phase) {
        this.cx = cx;
        this.cy = cy;
        this.r1 = r1;
        this.r2 = r2;
        this.ang = Math.random() * Math.PI * 2;
        this.speed = speed;
        this.phase = phase;
        this.dir = Math.random()<0.5?1:-1;
        this.col = randomColor();
        // Particle trail states
        this.particles = [];
        for(let i=0; i<SPIRO_PARTICLES; i++) {
            this.particles.push({phaseOffset: (i / SPIRO_PARTICLES) * Math.PI * 2});
        }
    }

    draw(t) {
        // Outer glow
        ctx.save();
        ctx.shadowColor = this.col;
        ctx.shadowBlur = 32;

        for (let i = 0; i < SPIRO_PARTICLES; i++) {
            const p = this.particles[i];
            // Spirograph parametric: 
            // https://en.wikipedia.org/wiki/Spirograph
            // R = this.r1, r = this.r2, a = phase
            let theta = this.ang + t * this.speed * this.dir + p.phaseOffset;
            let R = this.r1;
            let r = this.r2 + Math.sin(t*0.3 + this.phase) * 12;
            let O = 15 + 7 * Math.sin(t*0.7 + i+this.phase);
            let x = this.cx + (R - r) * Math.cos(theta) + O * Math.cos((R - r) * theta / r);
            let y = this.cy + (R - r) * Math.sin(theta) - O * Math.sin((R - r) * theta / r);

            // Animate color hue over time
            let baseHue = (t*25 + this.phase*100 + i*7) % 360;
            ctx.beginPath();
            ctx.arc(x, y, 7 + 4 * Math.sin(t + this.phase + i), 0, Math.PI*2);
            ctx.fillStyle = `hsla(${baseHue},98%,59%,0.42)`;
            ctx.fill();
        }

        ctx.restore();

        // Overlay a vivid, smaller "nucleus" particle at the end
        let theta = this.ang + t * this.speed * this.dir;
        let R = this.r1;
        let r = this.r2 + Math.sin(t*0.3 + this.phase) * 12;
        let O = 15 + 7 * Math.sin(t*0.7 + this.phase);
        let x = this.cx + (R - r) * Math.cos(theta) + O * Math.cos((R - r) * theta / r);
        let y = this.cy + (R - r) * Math.sin(theta) - O * Math.sin((R - r) * theta / r);
        ctx.save();
        ctx.beginPath();
        ctx.arc(x, y, 13 + 6*Math.abs(Math.cos(t*0.9 + this.phase)), 0, Math.PI*2);
        ctx.fillStyle = this.col;
        ctx.shadowColor = '#fff';
        ctx.shadowBlur = 15;
        ctx.globalAlpha = 0.88;
        ctx.fill();
        ctx.restore();
    }
}

function createSpiros() {
    spiros = [];
    let radius = Math.min(w, h) * 0.31;
    for (let i = 0; i < N_SPIRO; i++) {
        let angle = i * Math.PI * 2 / N_SPIRO + Math.sin(i*0.8);
        let cx = w/2 + radius * Math.cos(angle);
        let cy = h/2 + radius * Math.sin(angle);
        let r1 = 85 + 70 * Math.abs(Math.cos(i + angle));
        let r2 = 19 + 14*Math.sin(i);
        let speed = 0.15 + 0.1*Math.abs(Math.sin(i*0.9));
        let phase = Math.random()*Math.PI*2;
        spiros.push(new Spiro(cx, cy, r1, r2, speed, phase));
    }
}

// --- Background static fractal pulses ---
function drawFractalPulse(t) {
    ctx.save();
    const cx = w/2, cy = h/2;
    for (let k = 0; k < 6; k++) {
        let rad = Math.min(w, h) * (0.18 + 0.11*Math.sin(t*0.4 + k));
        let segments = 16 + 12*Math.sin(t*0.13 + k*1.3);
        ctx.globalAlpha = 0.13 + 0.07*Math.cos(t*0.4 + k*1.1);
        ctx.beginPath();
        for (let i = 0; i <= segments; i++) {
            let a = i/segments * Math.PI*2;
            let freq = 6 + 2*Math.cos(t*0.32 + k*1.6);
            let amp = 30 + 8*Math.sin(t*0.61 + k*1.9);
            let r = rad + Math.sin(a * freq + t*0.28 + k)*amp;
            let x = cx + r * Math.cos(a);
            let y = cy + r * Math.sin(a);
            if(i===0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        ctx.closePath();
        let baseHue = ((t*8 + k*45)%360);
        ctx.strokeStyle = `hsla(${baseHue},90%,61%,0.18)`;
        ctx.lineWidth = 7 + 4*Math.abs(Math.sin(t*0.5 + k));
        ctx.shadowColor = `hsl(${baseHue},80%,69%)`;
        ctx.shadowBlur = 27;
        ctx.stroke();
    }
    ctx.restore();
}

// --- Animation loop ---
let timeStart = null;

function animate(ts) {
    if (!timeStart) timeStart = ts;
    let t = (ts - timeStart)/900; // controls time scaling

    // Translucent background fade for trails/chromatic blending
    ctx.globalAlpha = 0.16;
    ctx.fillStyle = '#110421';
    ctx.fillRect(0,0,w,h);
    ctx.globalAlpha = 1.0;

    drawFractalPulse(t);

    // Vortex swirl rotation
    ctx.save();
    ctx.translate(w/2, h/2);
    ctx.rotate(0.13*Math.sin(t*0.28));
    ctx.translate(-w/2, -h/2);

    // Animate and draw all Spiros
    for (const spiro of spiros) {
        spiro.draw(t);
    }
    ctx.restore();

    requestAnimationFrame(animate);
}

// --- Interactivity: click to randomize ---
canvas.addEventListener('click', () => {
    createSpiros();
});

// Initialize & start
createSpiros();
animate();

</script>

<!--
* Psychedelic Spiro-Particle Vortex
* Click/tap to remix spiro orbits and color palette.
* Drifting neon, spirograph-style vortices with radiant trails and fractal pulses behind.
* Made with ♥ in generative JavaScript.
-->
```