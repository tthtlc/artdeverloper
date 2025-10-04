---
layout: fullscreen
title: "Psychedelic Sunflower Spirals"
tags:
  - graphics
---

<canvas id="canvas" width="800" height="600"></canvas>
<script>
(function() {
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    // Center of canvas
    const cx = canvas.width/2;
    const cy = canvas.height/2;

    // Parameters for the sunflower spiral
    const NUM_SEEDS = 450;
    const PHI = (Math.sqrt(5) + 1) / 2; // Golden ratio
    const BASE_RADIUS = 8;
    const SATURATION = 95;
    const LIGHTNESS = 60;

    let tick = 0;

    // For interactive "breeze"
    let wind = {x: 0, y: 0};

    canvas.addEventListener("mousemove", e => {
        const mx = e.offsetX - cx;
        const my = e.offsetY - cy;
        wind.x = mx * 0.00025;
        wind.y = my * 0.00025;
    });

    function draw() {
        // Psychedelic background with slow undulating color shift
        ctx.globalAlpha = 0.18;
        ctx.fillStyle = `hsl(${(tick/2)%360}, 70%, 12%)`;
        ctx.fillRect(0,0,canvas.width,canvas.height);
        ctx.globalAlpha = 1;

        for(let i=0; i<NUM_SEEDS; i++) {
            // Vogel's model for sunflower pattern
            const theta = i * Math.PI * 2 / (PHI*PHI);
            const r = BASE_RADIUS * Math.sqrt(i);

            // Animate spiral offset and "pulsing"
            const phase = theta + tick*0.015 + Math.sin(r/10 + tick*0.07)*0.25;

            // "Breezing" drift
            const wx = Math.sin(tick*0.01 + i)*wind.x*12;
            const wy = Math.cos(tick*0.0122 + i)*wind.y*12;

            // Wavy revolution
            const offsetRadius = r + 22*Math.sin(tick*0.02 + i*0.13);

            // Final coordinates
            const x = cx + Math.cos(phase)*offsetRadius + wx;
            const y = cy + Math.sin(phase)*offsetRadius + wy;

            // Animate hue for trailing rainbow effect
            const hue = (i*0.75 + tick*2 + 110*Math.sin(tick*0.007 + i*0.045))%360;

            // Animate petal size
            const sz = 9 + 7*Math.sin(tick*0.016 + i*0.11);
            const alpha = 0.54 + 0.4*Math.sin(tick*0.018 + i*0.14);

            ctx.save();
            ctx.translate(x, y);

            // Draw the "seed" - a wavy, multi-lobed petal/star
            ctx.beginPath();
            const lobes = 6 + Math.floor(3*Math.sin(tick*0.02 + i*0.201));
            for(let k=0; k<=lobes; k++) {
                const angle = 2*Math.PI*k/lobes;
                const amp = sz + Math.sin(angle*lobes + tick*0.1 + i*0.15)*sz*0.34;
                const px = Math.cos(angle) * amp;
                const py = Math.sin(angle) * amp;
                if(k===0) ctx.moveTo(px, py);
                else ctx.lineTo(px, py);
            }
            ctx.closePath();
            ctx.globalAlpha = alpha;
            ctx.fillStyle = `hsl(${hue},${SATURATION}%,${LIGHTNESS}%)`;
            ctx.shadowColor = `hsl(${(hue+100)%360},${SATURATION}%,30%)`;
            ctx.shadowBlur = 20;
            ctx.fill();

            // Inner circle "seed"
            ctx.beginPath();
            ctx.arc(0, 0, sz*0.45, 0, 2*Math.PI);
            ctx.globalAlpha = alpha*0.95;
            ctx.fillStyle = `hsl(${(hue+75)%360},90%,90%)`;
            ctx.shadowBlur = 6;
            ctx.shadowColor = `hsl(${(hue+200)%360},60%,35%)`;
            ctx.fill();
            ctx.restore();
        }
        tick++;
        requestAnimationFrame(draw);
    }

    // Initial dark background
    ctx.fillStyle = '#101014';
    ctx.fillRect(0,0,canvas.width,canvas.height);
    draw();
})();
</script>
