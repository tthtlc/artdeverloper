---
layout: fullscreen
title: "Hypnotic Lissajous Spirals"
tags:
  - graphics
---

<canvas id="lissajousCanvas" width="700" height="700"></canvas>
<script>
const canvas = document.getElementById('lissajousCanvas');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const W = canvas.width;
const H = canvas.height;

// Lissajous parameters
const curves = 8; // how many individual spirals
const ptsPerCurve = 360;
const spiralTurns = 7;

const colors = [
  '#FF0066', '#FFEA00', '#00FF99', '#00CFFF',
  '#7D00FF', '#FF34F6', '#FD5D47', '#39FF14'
];

function lerp(a, b, t) {
    return a + (b - a) * t;
}

// Particle objects tracing out Lissajous spirals
class SpiralParticle {
    constructor(i) {
        this.i = i;
        this.color = colors[i % colors.length];
        this.phase = (i / curves) * Math.PI * 2;
        this.freqA = 3 + (i % 5);
        this.freqB = 2 + ((i * 2) % 7);
        this.amp = lerp(W/8, W/2.5, Math.sin(i * Math.PI / curves));
        this.offset = Math.random() * Math.PI * 2;
    }

    getPoint(t, spiralPhase, radiusScale) {
        // Lissajous parametric equations with spiral radius
        let a = this.freqA;
        let b = this.freqB;
        let d = this.amp * radiusScale;
        let theta = t * Math.PI * 2 * spiralTurns + spiralPhase + this.offset;

        // Lissajous curve with spiral radius modulated
        // Add wave modulation for trippy effect
        const r = d * lerp(.35,1,Math.sin(theta*0.5 + this.phase)*0.5+0.5);

        return {
            x: W/2 + r * Math.sin(a*theta + this.phase),
            y: H/2 + r * Math.sin(b*theta),
        }
    }
}

// Particle system
let particles = [];
for (let i=0; i<curves; i++) {
    particles.push(new SpiralParticle(i));
}

let time = 0;
let prevImage;
const trailAlpha = 0.13;

function draw() {
    // Trippy fading trails
    ctx.globalAlpha = trailAlpha;
    ctx.fillStyle = "#070014";
    ctx.fillRect(0, 0, W, H);
    ctx.globalAlpha = 1;

    const spiralPhase = time * 0.2;
    const baseRadiusScale = lerp(0.7,1.2,0.5+0.5*Math.sin(time*0.07));

    for (let p = 0; p < particles.length; p++) {
        let particle = particles[p];

        ctx.save();
        ctx.beginPath();
        for (let i = 0; i <= ptsPerCurve; i++) {
            let t = i / ptsPerCurve;
            let phaseMod = Math.cos(time*0.06+particle.phase+t*Math.PI*2)*0.5+0.5;
            let radiusScale = baseRadiusScale * (0.9+0.13*phaseMod);

            let pt = particle.getPoint(t, spiralPhase, radiusScale);
            if (i === 0) {
                ctx.moveTo(pt.x, pt.y);
            } else {
                ctx.lineTo(pt.x, pt.y);
            }
        }
        // Rainbowish hue shift
        let hueShift = ((time*15 + p*48)%360);
        ctx.strokeStyle = `hsl(${hueShift}, 86%, 66%)`;
        ctx.shadowColor = colors[(p+1)%colors.length];
        ctx.shadowBlur = 22;
        ctx.lineWidth = 2.9 + 1.8*Math.sin(time*0.28+p);
        ctx.globalAlpha = .90;

        ctx.stroke();
        ctx.restore();
    }

    time += 0.012;
    requestAnimationFrame(draw);
}

// Smoother resize handler
window.addEventListener('resize', ()=>{
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
});
draw();

</script>
