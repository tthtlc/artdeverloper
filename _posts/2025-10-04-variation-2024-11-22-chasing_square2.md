---
layout: fullscreen
title: "Psychedelic Plasma Wave Rings"
tags:
  - graphics
---

<canvas id="plasmaRingsCanvas" width="500" height="500"></canvas>
<script>
const canvas = document.getElementById('plasmaRingsCanvas');
const ctx = canvas.getContext('2d');

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// Parameters
const CENTER_X = canvas.width / 2;
const CENTER_Y = canvas.height / 2;
const NUM_RINGS = 13;
const RING_BASE_RADIUS = Math.min(canvas.width, canvas.height) * 0.10;
const RING_SPACING = Math.min(canvas.width, canvas.height) * 0.04;
const POINTS_PER_RING = 180;
const WAVE_FREQS = [2.5, 3.8, 6.2];
const WAVE_SPEEDS = [1.3, 0.9, 2.5];

// Utility for color cycling
function hsl(h, s, l, a=1.0) {
    return `hsla(${h},${s}%,${l}%,${a})`;
}

// Animation function
function draw(time) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    const t = time * 0.001;

    for (let ring = 0; ring < NUM_RINGS; ring++) {
        const baseRadius = RING_BASE_RADIUS + RING_SPACING * ring;
        // Wave amplitude
        const amp = 24 + 16 * Math.sin(t * 0.45 + ring * 0.3);

        ctx.beginPath();
        for (let i = 0; i <= POINTS_PER_RING; i++) {
            const theta = (2 * Math.PI * i) / POINTS_PER_RING;

            // Dynamic distortion using multi-frequency sine waves
            let offset = 0;
            for (let w = 0; w < WAVE_FREQS.length; w++) {
                offset += 
                    Math.sin(
                        theta * WAVE_FREQS[w] +
                        t * WAVE_SPEEDS[w] +
                        ring * (0.75 + w * 0.4)
                    ) * (amp / (w + 2.3));
            }
            // Plasmatic noise
            offset += Math.sin(
                theta * 13 +
                t * 1.2 + 
                ring * 2
            ) * 5;

            const r = baseRadius + offset;
            const x = CENTER_X + r * Math.cos(theta);
            const y = CENTER_Y + r * Math.sin(theta);

            if (i === 0)
                ctx.moveTo(x, y);
            else
                ctx.lineTo(x, y);
        }

        // Paintbrush: Psychedelic evolving color
        const glowPhase = t * 0.7 + ring * 1.24;
        ctx.shadowColor = hsl(
            (110 + 48 * Math.sin(glowPhase)) % 360,
            90,
            70,
            0.45
        );
        ctx.shadowBlur = 21 + 14 * Math.sin(glowPhase+2);

        // Color shifts smoothly by time and ring index
        ctx.strokeStyle = hsl(
            (t * 60 + 120 + ring * 30) % 360,
            74 - 15 * Math.cos((t + ring) * 0.3),
            58 + 14 * Math.sin(t * 0.3 + ring * 0.15),
            0.96
        );
        ctx.lineWidth = 3.2 + 1.8 * Math.sin(t * 0.9 + ring);
        ctx.globalAlpha = 0.9 - ring * 0.018;

        ctx.stroke();
        ctx.shadowBlur = 0;
    }

    // "Plasma aura" background
    let grad = ctx.createRadialGradient(
        CENTER_X, CENTER_Y, 0, 
        CENTER_X, CENTER_Y, canvas.height/2.2
    );
    grad.addColorStop(0, hsl((t*40)%360,75,24,0.12));
    grad.addColorStop(0.7, hsl((t*18+80)%360,96,10,0.16));
    grad.addColorStop(1, hsl((t*20+160)%360,100,8,0.02));
    ctx.globalAlpha = 1;
    ctx.globalCompositeOperation = "lighter";
    ctx.fillStyle = grad;
    ctx.fillRect(0,0,canvas.width,canvas.height);
    ctx.globalCompositeOperation = "source-over";
    requestAnimationFrame(draw);
}

draw(0);

// Adapt canvas size
window.addEventListener('resize', ()=>{
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
});
</script>
