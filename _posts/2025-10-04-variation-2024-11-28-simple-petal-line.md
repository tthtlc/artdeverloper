---
layout: fullscreen
title: "HypnoWaves - Psychedelic Interference Fields"
tags:
  - graphics
---

<style>
canvas {
    background-color: #080826;
    border: 2px solid #333;
    display: block;
    margin: 40px auto;
    box-shadow: 0px 2px 24px #111b;
}
.equation {
    margin-top: 25px;
    font-size: 20px;
    font-style: italic;
    color: #f0f0ff;
    background: linear-gradient(90deg, #0ef 10%, #f0f 90%);
    padding: 8px 12px;
    border-radius: 6px;
    width: max-content;
    margin-left: auto;
    margin-right: auto;
}
.design-note {
    margin-top: 16px;
    font-size: 15px;
    color: #afcad7;
    text-align: center;
}
.parameter-labels {
    margin-top: 20px;
    font-size: 16px;
    color: #ff0fae;
    text-align: center;
}
</style>

<div class="equation">
    Field: z = Σ<sub>i=1</sub><sup>M</sup> A<sub>i</sub> · sin(f<sub>i</sub>·x + φ<sub>i</sub> + t) · cos(f<sub>i</sub>·y – t)
</div>

<div class="design-note">
    Watch waves and moiré hallucinations evolve endlessly. <br> Every few seconds, the field resets in new hypnotic harmony!
</div>

<div class="parameter-labels">
    Current Fields: M = <span id="field-count">3</span> | Palette: <span id="palette-label">Cosmic Candy</span>
</div>

<canvas id="hypnowave" width="720" height="720"></canvas>
<script>
const W = 720;
const H = 720;
const canvas = document.getElementById('hypnowave');
const ctx = canvas.getContext('2d');
const N = 240; // grid resolution per axis
let FIELD_COUNT = 3;
let PALETTE = 0;

const palettes = [
    {name: "Cosmic Candy", stops: [[0,"#12fff7"],[0.3,"#ff00e0"],[0.7,"#f5eb09"],[1,"#6100ef"]]},
    {name: "Lava Dream", stops: [[0,"#ff512f"],[0.4,"#dd2476"],[0.85,"#1500ff"],[1,"#3d008b"]]},
    {name: "Psy Wave", stops: [[0,"#00f5a0"],[0.6,"#ff0099"],[1,"#fff202"]]},
    {name: "Seafoam Pulse", stops: [[0,"#00d2ff"],[0.5,"#3a47d5"],[1,"#9ad3bc"]]},
    {name: "Mango Heat", stops: [[0,"#ffde00"],[0.3,"#fa2527"],[0.75,"#f357ff"],[1,"#00fff3"]]},
];

const fieldCountSpan = document.getElementById('field-count');
const paletteLabel = document.getElementById('palette-label');

// Utility to interpolate color stops
function makePalette(stops, N = 256) {
    // stops: array of [position (0-1), color string]
    function hexToRgb(hex) {
        hex = hex.replace("#","");
        if (hex.length==3)
            hex = hex.split("").map(x=>x+x).join("");
        const num = parseInt(hex,16);
        return [num>>16&255, num>>8&255, num&255];
    }
    let arr = [];
    for(let i = 0; i<stops.length-1; ++i){
        let [a,acol]=stops[i], [b,bcol]=stops[i+1];
        let ca = hexToRgb(acol), cb = hexToRgb(bcol);
        let start = Math.floor(a*N), end = Math.floor(b*N);
        let count = end-start;
        for(let k=0;k<=count;++k){
            let t = k/count;
            arr[start+k] = `rgb(${Math.round(ca[0]*(1-t)+cb[0]*t)},${Math.round(ca[1]*(1-t)+cb[1]*t)},${Math.round(ca[2]*(1-t)+cb[2]*t)})`;
        }
    }
    return arr;
}

// Function to generate a new random field descriptor
function newFields(m) {
    let fields = [];
    for (let i=0; i<m; ++i) {
        // Frequencies: mild irrational number to maximize interference patterns
        let fx = Math.random()*0.8 + 0.4 + Math.random()*2.7;
        let fy = Math.random()*0.8 + 0.4 + Math.random()*2.7;
        let amp = (Math.random()*0.17+0.21) * (Math.random()<0.2?2:1);
        let phase = Math.random() * Math.PI*2;
        fields.push({fx, fy, amp, phase, sign: Math.sign(Math.random()-0.5)||1});
    }
    return fields;
}

let fields = newFields(FIELD_COUNT);
let paletteArr = makePalette(palettes[PALETTE].stops, 512);

function renderField(t) {
    let img = ctx.createImageData(W,H);
    let dat = img.data;
    // Center: [-1,1] x [-1,1]
    for (let py=0; py<H; ++py){
        let y = (py-H/2) / (H/2); // -1 ... 1
        for (let px=0; px<W; ++px){
            let x = (px-W/2) / (W/2);
            let z = 0;
            for (let f=0; f<fields.length; ++f){
                let fd = fields[f];
                z += fd.amp * Math.sin(fd.fx * x + fd.phase + t*fd.sign)
                           * Math.cos(fd.fy * y - t*fd.sign);
            }
            // Normalize to 0...1 (with margin for constructive interference)
            let norm = (z + 1.6) / 3.2;
            let colIdx = Math.floor(norm * (paletteArr.length-1));
            let cstr = paletteArr[(colIdx+paletteArr.length)%paletteArr.length];
            // Parse rgb string
            let m = cstr.match(/\d+/g);
            dat[4*(py*W+px)]   = +m[0];
            dat[4*(py*W+px)+1] = +m[1];
            dat[4*(py*W+px)+2] = +m[2];
            dat[4*(py*W+px)+3] = 255;
        }
    }
    ctx.putImageData(img,0,0);
}

// Animate and reset logic
let t0 = Date.now();
let animating = true;
function animate() {
    if (!animating) return;
    let t = ((Date.now()-t0)/1100.0);
    renderField(t);
    requestAnimationFrame(animate);
}
animate();

function randomizeAll() {
    // Shuffle palette
    PALETTE = (PALETTE+1 + Math.floor(Math.random()*palettes.length))%palettes.length;
    FIELD_COUNT = 3 + Math.floor(Math.random()*3); // 3~5
    fields = newFields(FIELD_COUNT);
    paletteArr = makePalette(palettes[PALETTE].stops, 512);
    fieldCountSpan.textContent = FIELD_COUNT;
    paletteLabel.textContent = palettes[PALETTE].name;
}
setInterval(randomizeAll, 9000+Math.random()*4000);
randomizeAll();

// Pause/resume on click
canvas.onclick = function() {
    animating = !animating;
    if (animating) animate();
};
// Keyboard: space to randomize now; P to pause
window.addEventListener('keydown', e=>{
    if (e.key===' '){ randomizeAll(); }
    if (e.key.toLowerCase()==='p'){ animating = !animating; if (animating) animate(); }
});

// Prevent right-click context menu
window.addEventListener("contextmenu", e=>e.preventDefault());
</script>
