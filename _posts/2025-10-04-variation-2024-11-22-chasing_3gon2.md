```markdown
---
layout: fullscreen
title: "Shifting Waves of a Psychedelic Lissajous Garden"
tags:
  - graphics
---

Witness the hypnotic bloom of evolving Lissajous "flowers" dripping with color, forming a pulsating pattern garden. The lines dance along ever-morphing Lissajous curves, cycling through harmonic changes and a psychedelic palette.

<canvas id="lissajousGarden" width="600" height="600" style="background:black;display:block;margin:auto;max-width:98vw;max-height:98vh;"></canvas>
<script>
const canvas = document.getElementById('lissajousGarden');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;
let t = 0;

// Utility for smooth rainbow colors
function hsvToRgb(h, s, v) {
    let f = (n, k = (n + h / 60) % 6) => v - v * s * Math.max(Math.min(k,4-k,1),0);
    return [f(5)*255,f(3)*255,f(1)*255];
}

// Generate a list of center points for the flowers arranged in a spiral
function spiralCenters(num, cx, cy, rMin, rMax) {
    let arr = [];
    for (let i = 0; i < num; i++) {
        let frac = i/(num-1);
        let theta = Math.PI*2.2*frac;
        let r = rMin + (rMax-rMin)*frac;
        let x = cx + r * Math.cos(theta);
        let y = cy + r * Math.sin(theta);
        arr.push([x, y]);
    }
    return arr;
}

// Lissajous parametric curve
function lissajous(t, A, B, a, b, delta) {
    return [
        A * Math.sin(a * t + delta),
        B * Math.sin(b * t)
    ];
}

// Global animation
let flowers = [];

// Randomized Lissajous parameters for each flower
function setupFlowers() {
    flowers.length = 0;
    let nFlowers = 7 + Math.floor(Math.random()*4);
    let spiral = spiralCenters(nFlowers, W/2, H/2, 40, W*0.36 + Math.random()*40);
    for (let i = 0; i < nFlowers; i++) {
        // Each flower gets its own harmonic
        let A = 50 + 35*Math.random();
        let B = 50 + 35*Math.random();
        // Frequencies (odd/even, semi-coprime)
        let a = 2 + Math.floor(Math.random()*4); // 2-5
        let b = 3 + Math.floor(Math.random()*3);
        let phase = Math.random() * Math.PI*2;
        let cShift = Math.random() * Math.PI*2;
        flowers.push({
            center: spiral[i],
            A,B,a,b,
            phase, cShift,
            petals: 120 + Math.floor(Math.random()*60),
            baseColor: Math.random()
        });
    }
}
setupFlowers();

// Animate, evolving flower frequencies and phases over time
function draw() {
    // Psychedelic afterimage effect
    ctx.globalAlpha = 0.18;
    ctx.fillStyle = "rgba(0,0,16,0.22)";
    ctx.fillRect(0,0,W,H);
    ctx.globalAlpha = 1;
    
    let time = performance.now() * 0.00068;
    let tDrift = Math.sin(time*0.13)*0.6 + Math.sin(time*0.08)*0.3;

    for (let f=0; f<flowers.length; f++) {
        let fl = flowers[f];

        // Animate harmonics and phase slowly over time
        let a = fl.a + Math.sin(time*0.12 + fl.cShift)*0.6 + Math.cos(time*0.2 + f)*0.7;
        let b = fl.b + Math.cos(time*0.17 + fl.cShift)*0.5 + Math.sin(time*0.09 + f)*0.9;
        let phase = fl.phase + Math.cos(time*0.10 + fl.cShift)*0.7;

        // Animate size
        let scalePulse = 1 + Math.sin(time*0.47 + f)*0.11 + Math.sin(time*0.21 + f*1.2)*0.07;

        // Time offset for trailing motion
        let flowerTime = time + fl.cShift*0.8 + Math.sin(time*0.3+f)*0.11;

        // Draw multiple wavy outlines for trippy effect
        for (let outline=0; outline<5; outline++) {
            let wavy = 1 + 0.08 * outline + 0.10 * Math.sin(time*0.44 + outline);
            let strokeHue = (fl.baseColor + outline*0.105 + 0.1*flowerTime + tDrift*0.19) % 1;
            let pop = 0.71 + 0.23 * Math.sin(flowerTime + outline);
            let sat = 0.82, val = 1 - 0.13*outline;

            let [r,g,b] = hsvToRgb(strokeHue*360,sat,val*pop);
            ctx.strokeStyle = `rgba(${r|0},${g|0},${b|0},${0.33+0.13*outline})`;
            ctx.lineWidth = 1.2 + 0.7*outline;

            ctx.beginPath();
            for (let i=0;i<=fl.petals;i++) {
                let theta = Math.PI*2*i/fl.petals;
                let [x,y] = lissajous(
                    theta,
                    fl.A*scalePulse*wavy*(1+0.03*Math.sin(outline*1.8 + theta*3+flowerTime*0.6)),
                    fl.B*scalePulse*wavy*(1+0.025*Math.sin(outline*1.2 + theta*3.7+flowerTime*0.6)),
                    a, b, phase + outline*0.7
                );
                if (i===0)
                    ctx.moveTo(fl.center[0]+x, fl.center[1]+y);
                else
                    ctx.lineTo(fl.center[0]+x, fl.center[1]+y);
            }
            ctx.closePath();
            ctx.shadowColor = `rgba(${r|0},${g|0},${b|0},0.67)`;
            ctx.shadowBlur = 18+7*outline;
            ctx.stroke();
            ctx.shadowBlur = 0;
        }
    }
    t += 1;
    requestAnimationFrame(draw);
}

// Reseed flowers on click for variety
canvas.onclick = () => {
    setupFlowers();
};

draw();

</script>
```