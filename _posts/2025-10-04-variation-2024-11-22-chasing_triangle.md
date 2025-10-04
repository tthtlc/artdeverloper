---
layout: fullscreen
title: "Psychedelic Pulsing Spiral Waves"
tags:
  - graphics
---

<canvas id="psySpiralCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('psySpiralCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width;
const H = canvas.height;
const CX = W/2, CY = H/2;

const ARMS = 7;                   // Spiral arms
const WAVES = 26;                 // Wave repetitions per arm
const PARTICLES = 200;            // Points along spiral
const COLORS = 36;                // Color cycling
const PHASE_SPEED = 0.018;        // Overall motion speed

// Generate persistent wave offsets
let waveOffsets = [];
for(let i=0;i<ARMS;i++) {
  waveOffsets.push(Math.random()*Math.PI*2);
}

// Helper: HSV to RGB
function hsv(h,s,v) {
    let f=(n,k=(n+h/60)%6)=>v-v*s*Math.max(Math.min(k,4-k,1),0);
    return `rgb(${f(5)*255|0},${f(3)*255|0},${f(1)*255|0})`;
}

// Animation loop
let t = 0;
function draw() {
    ctx.globalAlpha = 0.13; // Trail effect
    ctx.fillStyle = "#0c0520";
    ctx.fillRect(0,0,W,H);
    ctx.globalAlpha = 1.0;
    t += PHASE_SPEED;

    for(let arm=0; arm<ARMS; arm++) {
        ctx.beginPath();
        for(let p=0; p<PARTICLES; p++) {
            let pct = p/(PARTICLES-1);

            // Spiral radius and angle computation
            let baseR = 80 + 240*pct; // Spread spiral from inner to outer
            let theta = (
                2*Math.PI*arm/ARMS           // arm rotation
                + 6*pct*2*Math.PI            // spiral winds
                + t*0.8                      // slow global rotation
            );

            // Wave modulations
            let localWave = Math.sin(
                WAVES*pct*2*Math.PI 
                + t*2 + waveOffsets[arm] + p*0.11
            );

            let pulse = 0.6+0.55*Math.sin(t*1.8 + waveOffsets[arm] + p*0.032);

            let r = baseR + 40*localWave*pulse + 17*Math.sin(pct*18 + t*2);
            
            let x = CX + Math.cos(theta) * r;
            let y = CY + Math.sin(theta) * r;

            if(p===0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        ctx.closePath();

        // Color cycling per arm, shifting with time
        let h = (360*arm/ARMS + t*38)%360;
        ctx.strokeStyle = hsv(h, 0.98, 0.99);

        ctx.shadowColor = hsv((h+38)%360, 0.9, 1);
        ctx.shadowBlur = 32;
        ctx.lineWidth = 2.5 + 2.5*Math.sin(t*2 + arm);
        ctx.stroke();
    }
    
    // Center pulse
    const centerGlow = 54 + 24*Math.sin(t*2);
    ctx.beginPath();
    ctx.arc(CX, CY, centerGlow, 0, 2*Math.PI);
    ctx.fillStyle = hsv((t*70)%360,0.66,1);
    ctx.globalAlpha = 0.4+0.3*Math.sin(t*3.2);
    ctx.shadowColor = hsv((t*68+120)%360,0.5,1);
    ctx.shadowBlur = 80 + 16*Math.sin(t*2.1);
    ctx.fill();
    ctx.globalAlpha = 1.0;
    ctx.shadowBlur = 0;

    requestAnimationFrame(draw);
}

// Responsive resize
function resize() {
    let s = Math.min(window.innerWidth,window.innerHeight)*0.97;
    canvas.width = canvas.height = s;
}
window.addEventListener('resize',()=>{
    resize();
});
resize();

// Start animation!
draw();
</script>
