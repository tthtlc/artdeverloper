```markdown
---
layout: fullscreen
title: "Psychedelic Rippling Spirals"
tags:
  - graphics
---

A hypnotic, psychedelic variation featuring colorful, evolving spiral waves constructed from many oscillating particles that reverberate out from the center. Use the controls to play with spiral count and wave distortion for endlessly shifting, shimmering patterns.

<style>
    .controls {
        margin: 15px 0;
        font-family: Arial, sans-serif;
        color: #fff;
        font-size: 1rem;
        background: rgba(0,0,0,0.3);
        border-radius: 8px;
        padding: 10px;
        width: max-content;
        box-shadow: 0 0 20px #0008;
    }
    .controls label {
        margin-right: 12px;
    }
    body {
        background: radial-gradient(ellipse at center, #110022 60%, #000014 100%);
    }
</style>
<div class="controls">
    <label for="spiralNum">Spirals: <span id="spiralVal">7</span></label>
    <input type="range" id="spiralNum" min="2" max="14" value="7">
    <label for="waveAmt">Wave distortion: <span id="waveVal">0.5</span></label>
    <input type="range" id="waveAmt" min="0" max="1.5" value="0.5" step="0.01">
</div>
<canvas id="psySpiral" style="background: transparent; display: block; margin:auto; border-radius:10px;" width="650" height="650"></canvas>
<script>
const canvas = document.getElementById('psySpiral');
const ctx = canvas.getContext('2d');

const spiralNumInput = document.getElementById('spiralNum');
const spiralValOut = document.getElementById('spiralVal');
const waveAmtInput = document.getElementById('waveAmt');
const waveValOut = document.getElementById('waveVal');

let spiralCount = parseInt(spiralNumInput.value); // Number of spirals
let waveAmt = parseFloat(waveAmtInput.value);     // Wave distortion

const W = canvas.width, H = canvas.height;
const cx = W/2, cy = H/2;

let tick = 0;

spiralNumInput.addEventListener('input', ()=>{
    spiralCount = parseInt(spiralNumInput.value);
    spiralValOut.textContent = spiralCount;
});
waveAmtInput.addEventListener('input', ()=>{
    waveAmt = parseFloat(waveAmtInput.value);
    waveValOut.textContent = waveAmt.toFixed(2);
});

// HSV-to-RGB helper
function HSVtoRGB(h, s, v){
    let f = (n, k = (h / 60 + n) % 6) => v - v*s*Math.max(Math.min(k,4-k,1),0);
    return [f(5)*255,f(3)*255,f(1)*255];
}

function draw(){
    ctx.clearRect(0,0,W,H);

    // Composite mode for glow
    ctx.globalCompositeOperation = 'lighter';

    // Ripple/spiral parameters
    const maxRad = W*0.41;
    const pointsPerSpiral = 350;
    const spiralSeparation = (2 * Math.PI) / spiralCount;
    const waveFreq = 6.2 + Math.sin(tick*0.002)*2.5; // Wave undulation

    for(let s=0; s<spiralCount; s++){
        let spiralAngle = s * spiralSeparation;
        for(let i=0; i<pointsPerSpiral; i++){
            let t = i/(pointsPerSpiral-1);
            const theta = spiralAngle + t*11*Math.PI + 
                Math.sin(tick*0.011+s*1.2+t*2.8)*0.11 + // global wiggle
                (waveAmt) * Math.sin(waveFreq*t*6 + tick*0.041 + s*1.7);
            const radius = t*maxRad +
                (30 + 82*Math.sin((tick*0.008) - s + t*3.9 + Math.cos(tick*0.03+t*9)))*waveAmt*
                Math.sin(waveFreq*t*2 + s + tick*0.1);

            // Particle position
            let x = cx + Math.cos(theta)*radius;
            let y = cy + Math.sin(theta)*radius;

            // Colorful cycling hue: rainbow, modulating by spiral and distance
            let hue = (tick*0.42 + 340*s/spiralCount + 340*t + 80*Math.sin(tick*0.015+s*1.8-t*11.1))%360;
            let sat = 0.89 - 0.5*t + 0.14*Math.sin(tick*0.021-s+t*3.1);
            let val = 0.83 + 0.2*Math.cos(hue*0.03+tick*0.05+t*7+s*2.6);

            let [r,g,b] = HSVtoRGB(hue, sat, val);
            ctx.beginPath();
            // Blob size, more near center
            let dotR = 2.5 + 3.5*(1-t)*(0.48+0.55*Math.sin(tick*0.03-t*2.77+s));
            ctx.arc(x,y,dotR,0,2*Math.PI);
            ctx.fillStyle = `rgba(${r|0},${g|0},${b|0},${0.17+0.32*(1-t)})`;
            ctx.fill();
        }
    }

    // Inner phosphorescent core
    let g = ctx.createRadialGradient(cx,cy,0,cx,cy,maxRad*0.20);
    g.addColorStop(0, "rgba(255,255,244,0.17)");
    g.addColorStop(0.4, "rgba(255,204,220,0.14)");
    g.addColorStop(1, "rgba(54,10,60,0.03)");
    ctx.globalCompositeOperation = "lighter";
    ctx.beginPath();
    ctx.arc(cx, cy, maxRad*0.20, 0, 2*Math.PI);
    ctx.fillStyle = g;
    ctx.fill();

    ctx.globalCompositeOperation = "source-over";
    tick++;
    requestAnimationFrame(draw);
}
draw();
</script>
```