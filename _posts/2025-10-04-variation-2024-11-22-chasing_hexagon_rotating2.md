---
layout: fullscreen
title: "Hypnotic Pulsating Spiral of Oscillating Waves"
tags:
  - graphics
---

<canvas id="spiralCanvas" width="600" height="600"></canvas>
<script>
const canvas = document.getElementById('spiralCanvas');
const ctx = canvas.getContext('2d');

// GLOBALS
const W = canvas.width;
const H = canvas.height;
const CENTER = {x: W/2, y: H/2};
const arms = 6;         // number of main spiral arms
const nodes = 180;      // nodes per spiral arm
const wavesPerArm = 3;  // how many secondary oscillations per spiral arm
const spiralTurns = 3;  // how many full turns the spiral makes
const spiralRadius = 220; // max spiral arm length

// COLOR PALETTE: Neon and ultra-vivid rainbow gradients
function hsv(h,s,v) {
    // h in [0, 1], s in [0, 1], v in [0,1]
    let i = Math.floor(h*6);
    let f = h*6 - i;
    let p = v*(1-s);
    let q = v*(1-s*f);
    let t = v*(1-s*(1-f));
    let r,g,b;
    switch(i%6){
        case 0: r=v,g=t,b=p;break;
        case 1: r=q,g=v,b=p;break;
        case 2: r=p,g=v,b=t;break;
        case 3: r=p,g=q,b=v;break;
        case 4: r=t,g=p,b=v;break;
        case 5: r=v,g=p,b=q;break;
    }
    return `rgb(${Math.round(r*255)},${Math.round(g*255)},${Math.round(b*255)})`;
}

// ANIMATION LOOP
let t = 0;
function animateSpiral() {
    ctx.clearRect(0, 0, W, H);

    // BACKGROUND
    // Subtle radial gradient background for extra depth
    let grad = ctx.createRadialGradient(CENTER.x, CENTER.y, 0, CENTER.x, CENTER.y, W/1.2);
    grad.addColorStop(0, "#100018");
    grad.addColorStop(1, "#1a002a");
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, W, H);

    // DRAW SPIRAL ARMS
    for (let arm=0; arm<arms; arm++) {
        ctx.save();
        ctx.translate(CENTER.x, CENTER.y);
        ctx.rotate((2*Math.PI/arms)*arm + t*0.21); // slow global swirl

        ctx.beginPath();

        for (let n=0; n<=nodes; n++) {
            // Spiral parameter [0..1]
            let f = n/nodes;
            // Main spiral curve: r increases nonlinearly for nice "logarithmic" spiral
            let angle = spiralTurns * 2 * Math.PI * f + Math.sin(f*8 + t*0.9 + arm)*0.09;
            let radius = spiralRadius * Math.pow(f, 0.86) * (0.95+0.07*Math.cos(f*5-t*0.8+arm));
            // Apply oscillation to radius for wave effect
            let wave =
                Math.sin(f*wavesPerArm*2*Math.PI + t*2 + arm*1.2) // main large amplitude wave
                * (10 + 12*Math.sin(t + arm + f*8))                // modulated amplitude
                + Math.sin(t*2.1 + f*11 + arm*0.5)*6               // secondary wobble
            ;
            let x = Math.cos(angle) * (radius + wave);
            let y = Math.sin(angle) * (radius + wave);

            if (n===0) {
                ctx.moveTo(x, y);
            } else {
                ctx.lineTo(x, y);
            }
        }

        // Neon stroke
        let color = hsv(
            (arm/arms + t*0.07 + Math.sin(t/2 + arm)*0.16) % 1, // cycling color hue
            0.9 - 0.1*Math.sin(t + arm), 
            1
        );
        ctx.shadowColor = color;
        ctx.shadowBlur = 12+10*Math.abs(Math.sin(t*2+arm));
        ctx.lineWidth = 3.6 + 1.2*Math.sin(t+arm*0.7);
        ctx.strokeStyle = color;
        ctx.stroke();
        ctx.restore();
    }

    // PULSE DOTS Traveling Along the Spirals (micro "comets")
    for (let i=0; i<arms; i++) {
        let dotF = (t*0.18 + i/arms) % 1;                      // each travels at different speed
        let baseAngle = spiralTurns * 2 * Math.PI * dotF;
        let baseRadius = spiralRadius * Math.pow(dotF, 0.86);
        let angle = baseAngle + Math.sin(dotF*9 + t*0.6 + i)*0.09;
        let radius = baseRadius * (0.96+0.07*Math.cos(t+dotF*5+i));
        // Trail waves
        let wave = Math.sin(dotF*wavesPerArm*2*Math.PI + t*2 + i*1.2)*13;
        let px = CENTER.x + Math.cos(angle) * (radius + wave);
        let py = CENTER.y + Math.sin(angle) * (radius + wave);

        ctx.beginPath();
        ctx.arc(px, py, 11 + 3*Math.sin(t*1.7 + i), 0, 2*Math.PI);
        ctx.fillStyle = hsv((dotF + t*0.2 + i/arms)%1, 1, 1);
        ctx.shadowColor = "#fff";
        ctx.shadowBlur = 28 + 8*Math.sin(t+i);
        ctx.globalAlpha = 0.88;
        ctx.fill();
        ctx.globalAlpha = 1;
        ctx.shadowBlur = 0;
    }

    // Center black hole
    ctx.beginPath();
    ctx.arc(CENTER.x, CENTER.y, 30 + 10*Math.abs(Math.sin(t*0.7)), 0, 2*Math.PI);
    ctx.fillStyle = "#040013";
    ctx.shadowBlur = 35;
    ctx.shadowColor = "#22ddff";
    ctx.fill();
    ctx.shadowBlur = 0;

    t += 0.028; // Animation speed

    requestAnimationFrame(animateSpiral);
}

// Start
animateSpiral();

// Make it crispy on HiDPI
if(window.devicePixelRatio>1){
    let oldW = canvas.width, oldH = canvas.height;
    canvas.width = oldW*2;
    canvas.height = oldH*2;
    canvas.style.width = oldW+"px";
    canvas.style.height = oldH+"px";
    ctx.setTransform(2,0,0,2,0,0);
}
</script>
