---
layout: fullscreen
title: "Hypnotic Infinite Sine Blossom"
tags:
  - graphics
---

A mesmerizing hypnotic animation of endlessly blooming, morphing sine-blossoms. Petals spiral, undulate, and radiate with high-chroma colors, evoking psychedelic trance and infinite flow.  
Pure JavaScript generative art—edit any parameter for new visual surprises.

<canvas id="sineBlossomCanvas" width="800" height="800" style="width:100%; max-width:900px; background:#111; display:block; margin:40px auto; border-radius:30px;"></canvas>
<script>
const canvas = document.getElementById('sineBlossomCanvas');
const ctx = canvas.getContext('2d');
const w = canvas.width, h = canvas.height;
const cx = w / 2, cy = h / 2;

function lerp(a, b, t) {
  return a + (b - a) * t;
}

// Utility: convert HSV to RGB
function hsv2rgb(h, s, v) {
  let f = n => {
    let k = (n + h/60) % 6;
    return v - v * s * Math.max(Math.min(k,4-k,1),0);
  }
  return [f(5),f(3),f(1)];
}

// Draw one blossom/petal structure
function drawBlossom(time, baseRadius, petals, spiralFreq, colorOffset, petalSpread, wobbleAmp) {
    ctx.save();
    ctx.translate(cx, cy);

    const steps = 500;
    ctx.beginPath();
    for (let i = 0; i <= steps; i++) {
        let t = i / steps;
        let angle = t * Math.PI * 2 * spiralFreq;
        
        // The "bloom" radius is modulated by several sine/cosine terms
        let r = baseRadius 
            + Math.sin(t * petals * Math.PI * 2 + time * 0.8 + colorOffset) * petalSpread
            + Math.cos(angle * 1.5 + time * 1.6 - colorOffset*2) * wobbleAmp
            + Math.sin(time*0.3 + t*13) * lerp(0, 20, Math.sin(time+t*3))
            + lerp(0, 30, Math.sin(time*0.78+colorOffset*2.2) * Math.sin(t*spiralFreq));
        let x = Math.cos(angle) * r;
        let y = Math.sin(angle) * r;
        if (i===0) ctx.moveTo(x,y);
        else ctx.lineTo(x,y);
    }
    // Color: trippy shifting
    let baseHue = (Math.sin(time*0.13 + colorOffset) * 60 + time*28 + colorOffset*200) % 360;
    let sat = lerp(0.7, 1, Math.sin(time*0.53+colorOffset*2.5));
    let val = lerp(0.6,1,Math.cos(time*0.73-colorOffset*4));
    let [r,g,b] = hsv2rgb(baseHue, sat, val);
    ctx.shadowColor = `rgba(${Math.floor(r*255)},${Math.floor(g*255)},${Math.floor(b*255)},0.85)`;
    ctx.shadowBlur = 22;
    ctx.strokeStyle = `rgba(${Math.floor(r*255)},${Math.floor(g*255)},${Math.floor(b*255)},0.77)`;
    ctx.lineWidth = 2 + Math.sin(time*0.7+colorOffset)*1.3;
    ctx.stroke();
    ctx.restore();
}

// Main animation loop
function animate(t) {
    t *= 0.001; // ms to seconds
    // Black background fade for trails
    ctx.globalAlpha = 0.18;
    ctx.fillStyle = '#111';
    ctx.fillRect(0,0,w,h);
    ctx.globalAlpha = 1;
    
    // Multiple overlapping blossoms with distinct parameters
    for (let i = 0; i < 7; i++) {
        // Animate all parameters for a living, evolving feel
        let anglePhase = t*0.33 + i*0.74;
        let baseRad = lerp(120, 210, Math.sin(anglePhase+Math.sin(t*0.19+i)));
        let petals = lerp(5, 11, Math.sin(anglePhase*1.5 - i*t*0.07));
        let spiralFreq = lerp(3, 7, Math.cos(anglePhase*0.8 + t*i*0.09));
        let petalSpread = lerp(24, 55, Math.sin(anglePhase*2 + t*0.6));
        let wobbleAmp = lerp(18,42, Math.cos(i*0.8 + t*0.99));
        let colorOffset = i * 0.333 + Math.sin(t*0.31 + i);
        drawBlossom(t, baseRad, petals, spiralFreq, colorOffset, petalSpread, wobbleAmp);
    }

    // Central rotating dot cluster: throbbing, breathing
    ctx.save();
    ctx.translate(cx, cy);
    let dots = 26, rot = t*0.38;
    for(let i=0; i<dots; i++) {
        let a = i * Math.PI*2/dots + rot;
        let rad = lerp(38, 62, Math.sin(t*0.57+i*0.3));
        let px = Math.cos(a) * rad;
        let py = Math.sin(a) * rad;
        let dotSize = lerp(4,8,Math.sin(t*0.6 + i));
        let hue = (t*70+i*30)%360;
        let [r,g,b] = hsv2rgb(hue, 0.95, 1);
        ctx.beginPath();
        ctx.arc(px,py,dotSize,0,Math.PI*2);
        ctx.globalAlpha = 0.75;
        ctx.fillStyle = `rgb(${Math.floor(r*255)},${Math.floor(g*255)},${Math.floor(b*255)})`;
        ctx.shadowColor = ctx.fillStyle;
        ctx.shadowBlur = 17;
        ctx.fill();
    }
    ctx.globalAlpha = 1;
    ctx.restore();

    requestAnimationFrame(animate);
}

// Responsive: auto-resize the canvas cleanly
function resizeCanvas() {
    const size = Math.min(window.innerWidth, window.innerHeight, 900);
    canvas.width = canvas.height = size;
}
resizeCanvas();
window.addEventListener('resize', ()=>{
    resizeCanvas();
    // Clear everything when resizing
    ctx.clearRect(0,0,canvas.width,canvas.height);
});

animate(0);
</script>
