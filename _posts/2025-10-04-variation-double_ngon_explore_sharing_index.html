---
layout: fullscreen
title: Psychedelic Möbius Flowfield
tags:
  - graphics
---

<style>
        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background: radial-gradient(ellipse farthest-corner at 50% 50%, #000019 0%, #310024 100%);
        }
        canvas {
            background: radial-gradient(circle at 50% 50%, #141e30 0%, #243b55 100%);
            border: 1px solid #320032;
            box-shadow: 0 2px 30px #980dc6aa, 0 0 1px #000;
            margin-bottom: 28px;
        }
        .controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            background: #121015bb;
            border-radius: 12px;
            padding: 18px 30px 12px 30px;
            color: #fff;
            margin-bottom: 10px;
            box-shadow: 0 1px 8px #a140fb22;
        }
        .controls label {
            margin-right: 10px;
            font-weight: 500;
        }
        .controls input[type="range"] {
            accent-color: #e21fff;
            margin-bottom: 10px;
        }
        .controls span {
            min-width: 32px;
            display: inline-block;
            text-align: left;
        }
        .buttons {
            margin-top: 14px;
        }
        .capture-btn {
            padding: 10px 22px;
            background-image: linear-gradient(90deg, #7D26FF, #CB4EFF);
            color: white;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            font-size: 16px;
            margin-bottom: 8px;
            font-weight: bold;
            box-shadow: 0 1px 10px #cb4eff33;
        }
        .capture-btn:active {
            background-image: linear-gradient(90deg, #7800b3, #b148b8);
        }
        .share-links a {
            display: block;
            margin: 4px;
            text-decoration: none;
            color: #a147ff;
            font-size: 15px;
            font-weight: 600;
        }
        .share-links a:hover {
            color: #e21fff;
        }
</style>
<canvas id="animationCanvas" width="600" height="600"></canvas>
<div class="controls">
        <div>
            <label for="mobiusTwist">Möbius twist:</label>
            <input type="range" id="mobiusTwist" name="mobiusTwist" min="0" max="2" step="0.01" value="1">
            <span id="mobiusTwistValue">1.00</span>
        </div>
        <div>
            <label for="stripeCount">Stripe count:</label>
            <input type="range" id="stripeCount" name="stripeCount" min="4" max="32" step="1" value="18">
            <span id="stripeCountValue">18</span>
        </div>
        <div>
            <label for="trailFade">Trail fade:</label>
            <input type="range" id="trailFade" name="trailFade" min="0.02" max="0.20" step="0.01" value="0.08">
            <span id="trailFadeValue">0.08</span>
        </div>
    </div>
    <div class="buttons">
        <button class="capture-btn" id="captureBtn">Capture PNG</button>
        <div class="share-links">
            <a id="shareFacebook" href="#" target="_blank">Share on Facebook</a>
            <a id="shareWhatsApp" href="#" target="_blank">Share on WhatsApp</a>
        </div>
    </div>
    <script>
document.addEventListener("contextmenu", function(event) { event.preventDefault(); });
const canvas = document.getElementById('animationCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width, H = canvas.height;
const CX = W / 2, CY = H / 2;

let mobiusTwist = 1.00;
let stripeCount = 18;
let trailFade = 0.08;

// Controls
const mobiusTwistSlider = document.getElementById('mobiusTwist');
const mobiusTwistValue = document.getElementById('mobiusTwistValue');
const stripeCountSlider = document.getElementById('stripeCount');
const stripeCountValue = document.getElementById('stripeCountValue');
const trailFadeSlider = document.getElementById('trailFade');
const trailFadeValue = document.getElementById('trailFadeValue');

mobiusTwistSlider.addEventListener('input', e=>{
    mobiusTwist = parseFloat(e.target.value);
    mobiusTwistValue.textContent = mobiusTwist.toFixed(2);
});
mobiusTwistValue.textContent = mobiusTwist.toFixed(2);

stripeCountSlider.addEventListener('input', e=>{
    stripeCount = parseInt(e.target.value);
    stripeCountValue.textContent = stripeCount;
});
stripeCountValue.textContent = stripeCount;

trailFadeSlider.addEventListener('input', e=>{
    trailFade = parseFloat(e.target.value);
    trailFadeValue.textContent = trailFade.toFixed(2);
});
trailFadeValue.textContent = trailFade.toFixed(2);

// --- Möbius strip parameterization & animation

// Möbius strip surface (twice-around interval u ∈ [0, 2*PI])
function mobiusPoint(u, v, twist) {
    // u: angle along loop [0, 2*PI]
    // v: position across width [-1, 1]
    // twist: float >=0, how multi-twisted the surface is
    // Returns {x, y, z}
    // Large radius
    const R = 185;
    // Strip width
    const w = 48;
    // Number of "twists"
    // By modulating phi, we can do fractional twists for more visual play!
    const phi = (twist * u / 2);
    // Mobius: moves v along the cross-section, and orients the cross-section with half-twist!
    const x = (R + w * v * Math.cos(phi)) * Math.cos(u);
    const y = (R + w * v * Math.cos(phi)) * Math.sin(u);
    const z = w * v * Math.sin(phi);
    return {x, y, z};
}

// Project 3D (orthographic, with "fake" rotation for trippiness)
function project3D(pt, t) {
    // t ∈ [0,∞] is time
    // axis rotation
    const angleY = t * 0.25; // Slowly rotate
    const angleZ = Math.sin(t * 0.15) * 1.25;
    // Matrix rotations
    let {x, y, z} = pt;
    // rotZ
    let xz =  x * Math.cos(angleZ) - y * Math.sin(angleZ);
    let yz =  x * Math.sin(angleZ) + y * Math.cos(angleZ);
    x = xz; y = yz;
    // rotY
    let xx =  x * Math.cos(angleY) + z * Math.sin(angleY);
    let zz = -x * Math.sin(angleY) + z * Math.cos(angleY);
    x = xx; z = zz;
    // Center
    return {
        x: x + CX,
        y: y + CY - 15 // vertically center
    };
}

// Stripe color palette, psychedelic RYB rainbow
function colorByStripe(stripeIdx, totalStripes, t, uNorm) {
    // Animate the full palette in a shifting color wheel
    const offset = t * 0.11 + stripeIdx * Math.PI / 10;
    let hue = ((stripeIdx / totalStripes) + uNorm + Math.sin(offset)*0.07 + offset/10) % 1.0;
    if (hue < 0) hue += 1;
    hue = (hue * 360 + 320) % 360; // Shift overall palette magenta
    // Pulse luminance for shape
    const lum = 68 + 30 * Math.sin(uNorm * 2 * Math.PI + t * 1.2 + stripeIdx);
    const sat = 88 + 10 * Math.cos(uNorm * 12 * Math.PI + stripeIdx);
    return `hsl(${hue},${sat}%,${lum}%)`;
}

// Main animated stripes
const stripes = [];
const MESH_U = 340; // points along the loop
const STRIPE_VS = 17; // points across stripe width
function setupStripes() {
    stripes.length = 0; // Reset
    for (let s = 0; s < stripeCount; ++s) {
        // Each stripe exists on a parallel path along the Möbius strip
        let v = -1 + 2 * (s + 0.5) / stripeCount; // from -1 to 1
        stripes.push({
            v: v, // stripe path
            width: 2.5 + 1.5*Math.abs(Math.sin(s*17.1)), // animate width variation
        });
    }
}
setupStripes();

// Animate ("reset" stripes on count change)
stripeCountSlider.addEventListener('input', setupStripes);

let time = 0;

// Animate trail overlays
function fadeCanvas() {
    ctx.save();
    ctx.globalAlpha = trailFade;
    ctx.fillStyle = "#110112";
    ctx.fillRect(0,0,W,H);
    ctx.restore();
}

function drawMöbiusStripes(t) {
    fadeCanvas();

    // Animate twist
    const twist = mobiusTwist + 0.09*Math.sin(t * 0.37);

    // For shading/highlight
    const light = {x: 220, y: -350, z: 270};

    for (let s = 0; s < stripes.length; ++s) {
        const {v, width} = stripes[s];
        // For hue
        for (let ui = 0; ui < MESH_U; ++ui) {
            let u = ui / (MESH_U-1) * 2 * Math.PI; // goes from 0 to 2π
            const uNorm = ui/(MESH_U-1);
            // Across stripe thickness, for 3D-looking edges
            for (let vi = 0; vi < STRIPE_VS; ++vi) {
                let dv = (vi - (STRIPE_VS-1)/2) * (width/STRIPE_VS) * 0.98;
                // Animate with subtle ribbon motion
                let wave = Math.sin(u * (6+1.85*s) + v * 3 + s + t*0.92 + Math.sin(s*2.5))*0.16;
                let pt3 = mobiusPoint(u + wave, v + dv, twist);
                // Lighting: For Möbius, normal can be approximated by dv offset
                let pt3n = mobiusPoint(u + wave, v + dv + 0.014, twist);
                let nx = pt3n.x - pt3.x, ny = pt3n.y - pt3.y, nz = pt3n.z - pt3.z;
                let nlen = Math.sqrt(nx*nx + ny*ny + nz*nz)+1e-6;
                nx/=nlen; ny/=nlen; nz/=nlen;
                // Dot light
                let lx = light.x - pt3.x, ly = light.y - pt3.y, lz = light.z - pt3.z;
                let llen = Math.sqrt(lx*lx+ly*ly+lz*lz)+1e-6;
                lx/=llen; ly/=llen; lz/=llen;
                let dot = (nx*lx+ny*ly+nz*lz)*0.7+0.6;
                dot = Math.pow(dot,0.7);

                // 2d projection
                let pt2 = project3D(pt3, t);

                // Dynamic color, w/ shading and glow
                ctx.save();
                ctx.globalAlpha = 1;
                ctx.beginPath();
                ctx.arc(pt2.x, pt2.y, 1.45 + 0.45*dot, 0, 2*Math.PI);

                ctx.shadowBlur = 6 + 4.5 * dot + 2*(Math.sin(u*7 + s) + 1);
                ctx.shadowColor = colorByStripe(s, stripes.length, t, uNorm);

                ctx.fillStyle = colorByStripe(s, stripes.length, t, uNorm);
                ctx.filter = "blur(0.5px)";

                ctx.globalAlpha = 0.95 * (0.45 + 0.65*dot);

                ctx.fill();
                ctx.restore();
            }
        }
    }

    // Subtle Möbius edge highlight (drawn above)
    ctx.save();
    ctx.globalAlpha = 0.48;
    ctx.beginPath();
    for(let ui = 0; ui < MESH_U; ++ui) {
        let u = ui / (MESH_U-1) * 2 * Math.PI;
        let edge = mobiusPoint(u, 1, twist);
        let p2 = project3D(edge, t);
        if(ui===0) ctx.moveTo(p2.x, p2.y); else ctx.lineTo(p2.x, p2.y);
    }
    ctx.strokeStyle = "rgba(255,255,255,0.24)";
    ctx.lineWidth = 3;
    ctx.shadowBlur = 5;
    ctx.shadowColor = "#fff";
    ctx.stroke();
    ctx.restore();
}

// Animate psychedelic Möbius
function animate() {
    time += 0.0145;
    drawMöbiusStripes(time);
    requestAnimationFrame(animate);
}

// Initial fade-in to clear dirty pixels
ctx.fillStyle = "#000026";
ctx.fillRect(0,0,W,H);

setTimeout(animate, 80);

// -- PNG capture & sharing functions

document.getElementById('captureBtn').addEventListener('click', function () {
    // Temporarily clear fade for static PNG
    const tempFade = trailFade;
    trailFade = 1.0;
    drawMöbiusStripes(time);
    setTimeout(()=>{
        const dataURL = canvas.toDataURL('image/png');
        const link = document.createElement('a');
        link.href = dataURL;
        link.download = 'psychedelic_mobius.png';
        link.click();
        trailFade = tempFade;
    }, 40);
});

document.getElementById('shareFacebook').addEventListener('click', function () {
    // Share PNG as dataurl
    const dataURL = canvas.toDataURL('image/png');
    const facebookShareUrl = `https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(dataURL)}`;
    this.href = facebookShareUrl;
});

document.getElementById('shareWhatsApp').addEventListener('click', function () {
    const dataURL = canvas.toDataURL('image/png');
    const whatsappShareUrl = `https://api.whatsapp.com/send?text=${encodeURIComponent(dataURL)}`;
    this.href = whatsappShareUrl;
});
</script>
