---
layout: fullscreen
title: "Hypno-Plasma Web: Animated Psychedelic Wave Tiling"
tags:
  - graphics
---

<style>
canvas {
    display: block;
    margin: 30px auto;
    background: black;
    border: 2px solid #3ce3ff;
}
#webControls {
    width: 520px; 
    margin: 0 auto; 
    text-align: center;
    font-size: 17px;
    color: #33ffee;
    font-family: "Fira Mono", "Courier New", monospace;
    letter-spacing: 1px;
}
input[type=range] {
    width: 130px;
}
</style>
<div id="webControls">
  <label for="freqRange">Wave Tiles: </label>
  <input type="range" id="freqRange" min="3" max="12" value="7" oninput="updateFreq()"><span id="freqValue">7</span>
  &nbsp;&nbsp;
  <label for="ampRange">Distort: </label>
  <input type="range" id="ampRange" min="10" max="120" value="40" oninput="updateAmp()"><span id="ampValue">40</span>
</div>
<canvas id="plasmaCanvas" width="600" height="600"></canvas>

<script>
const canvas = document.getElementById('plasmaCanvas');
const ctx = canvas.getContext('2d');
const freqRange = document.getElementById('freqRange');
const ampRange = document.getElementById('ampRange');
const freqValue = document.getElementById('freqValue');
const ampValue = document.getElementById('ampValue');

let freq = parseInt(freqRange.value, 10);
let amp = parseInt(ampRange.value, 10);
let time = 0;

// Generate color
function getColor(x, y, t) {
    // Mix three sine plasma fields
    let v =
        0.6 + 0.4 * Math.sin(0.11 * t + x * 0.07 + y * 0.11) +
        0.6 + 0.4 * Math.cos(0.07 * t + x * 0.17 - y * 0.05) +
        0.6 + 0.4 * Math.sin(0.14 * t + (x * x + y * y) * 0.00008);
    v /= 3;
    const hue = 180 + 180 * v + 50 * Math.sin((y+time)*0.008); // rainbow + wave
    const sat = 82 + 18 * Math.cos(x * 0.011 + t * 0.03);
    const light = 48 + 38 * Math.sin(0.0007 * x * y + t*0.002);
    return `hsl(${hue}, ${sat}%, ${light}%)`;
}

function drawWeb(freq, amp, t) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    const w = canvas.width;
    const h = canvas.height;
    const tileW = w/(freq+0.5);
    const tileH = h/(freq+0.5);

    // Draw tiled wave webs with curved lines and shifting color
    for(let i=0; i<=freq; i++) {
        for(let j=0; j<=freq; j++) {
            // Center of this tile
            const cx = i * tileW + tileW/2;
            const cy = j * tileH + tileH/2;

            let phase = Math.sin(t*0.007 + cx*0.01 + cy*0.01);
            let waviness = (amp + 22*Math.sin(t*0.005+cx*0.009) + 9*Math.cos(cy*0.012 + t*0.03));

            // Draw a waving 'spiderweb'
            ctx.save();
            ctx.translate(cx,cy);
            let numRings = 7+Math.floor(3*Math.sin(i*0.4 + j*0.5 + t*0.008));
            let numArms = 8+Math.floor(2*Math.cos(j*0.7 + t*0.012 - i*0.8));
            for(let r=1; r<=numRings; r++) {
                ctx.beginPath();
                for(let k=0; k<=numArms; k++) {
                    let a = (2 * Math.PI / numArms) * k;
                    let radius =
                        r * tileW/(2.8*numRings) +
                        waviness * Math.sin(phase + r*0.7 + a*3.12 + Math.cos(a*2 + t*0.007 + r*0.14));
                    let x = radius * Math.cos(a);
                    let y = radius * Math.sin(a);
                    if(k==0) ctx.moveTo(x,y);
                    else ctx.lineTo(x,y);
                }
                let p = 0.66*r/numRings + 0.37*i/freq + 0.31*j/freq;
                ctx.strokeStyle = getColor(cx+cx*p, cy+cy*p, t+80*r);
                ctx.globalAlpha = 0.82 - 0.05*r;
                ctx.lineWidth = 2 + 1.5*Math.sin(a + t*0.0063 + r*0.7);
                ctx.shadowColor = 'cyan';
                ctx.shadowBlur = 8 + 4*Math.sin(t*0.01+r*0.9);
                ctx.stroke();
            }
            // Draw arms as Bezier curves to midpoint of next tiles (psychedelic "plasma" links!)
            for(let k=0; k<numArms; k++) {
                let a = (2 * Math.PI / numArms) * k + Math.sin(j*0.8 + i*0.8 + t*0.01);
                let radius = tileW/2.1 + 0.3*waviness*Math.sin(a*freq*0.19+t*0.006+r);
                let x = radius * Math.cos(a);
                let y = radius * Math.sin(a);
                // Target tile: wrap edge
                let ni = (i + Math.round(Math.sin(a+t*0.02))) % (freq+1);
                let nj = (j + Math.round(Math.cos(a-t*0.03))) % (freq+1);
                if(ni<0) ni=freq; if(nj<0) nj=freq;
                let tx = ni * tileW + tileW/2 - cx;
                let ty = nj * tileH + tileH/2 - cy;
                ctx.beginPath();
                ctx.moveTo(0,0);
                // Calculate bezier control midway out, warped by time
                let mx = x*0.7 + tileW*0.17 * Math.sin(t*0.02 + a*2);
                let my = y*0.7 + tileH*0.17 * Math.cos(t*0.016 - a*3);
                ctx.bezierCurveTo(mx, my, tx*0.6, ty*0.6, tx, ty);
                ctx.strokeStyle = getColor(cx+x, cy+y, t+900*k);
                ctx.globalAlpha = 0.31 + 0.25*((Math.sin(a+t*0.002+cx*0.004+cy*0.003)+1)/2);
                ctx.lineWidth = 1.1 + 1.2*((Math.cos(k+t*0.009)+1)/2);
                ctx.shadowBlur = 14;
                ctx.shadowColor = '#fff';
                ctx.stroke();
            }
            ctx.restore();
        }
    }
}

function animate() {
    time += 1.6;
    drawWeb(freq, amp, time);
    requestAnimationFrame(animate);
}

function updateFreq() {
    freq = parseInt(freqRange.value, 10);
    freqValue.textContent = freq;
}
function updateAmp() {
    amp = parseInt(ampRange.value, 10);
    ampValue.textContent = amp;
}

// Initial GUI values
freqValue.textContent = freq;
ampValue.textContent = amp;

animate();
</script>
