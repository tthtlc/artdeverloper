```markdown
---
layout: fullscreen
title: "Hypnotic Waveform Spirals"
tags:
  - graphics
---

A mesmerizing exploration of evolving spiral waveforms. Here, radiating arms of a spiral pulse and twist with time, creating psychedelic patterns reminiscent of both fractals and liquid moirés. Each point oscillates with a different frequency and color, creating interference effects and color bursts.

<canvas id="spiralWaveCanvas" width="600" height="600" style="background:black;display:block;margin:0 auto;"></canvas>
<script>
const canvas = document.getElementById('spiralWaveCanvas');
const ctx = canvas.getContext('2d');

const W = canvas.width;
const H = canvas.height;
const centerX = W / 2;
const centerY = H / 2;

const ARMS = 9;         // Number of main spiral arms
const PTS_PER_ARM = 130; // Number of points along each arm
const WAVE_COUNT = 5;    // Number of overlapping waveforms per arm
const SPIRAL_TURNS = 2.5; // Number of spiral rotations
const BASE_RADIUS = 60;   // Where arms start from center
const MAX_RADIUS = 220;   // How far the arms go out

function lerp(a, b, t) {
    return a + (b - a) * t;
}

function hsv2rgb(h, s, v, a=1.0) {
    let f = (n, k = (h / 60 + n) % 6) => v - v * s * Math.max(Math.min(k,4-k,1),0);
    return `rgba(${f(5)*255|0},${f(3)*255|0},${f(1)*255|0},${a})`;
}

let lastTimestamp = 0;

function animate(ts) {
    ctx.clearRect(0,0,W,H);
    let time = ts * 0.001; // seconds
    let spiralAngleSpread = SPIRAL_TURNS * 2 * Math.PI;

    for (let arm = 0; arm < ARMS; arm++) {
        let armPhase = (arm/ARMS) * 2 * Math.PI;
        ctx.beginPath();
        for (let i = 0; i < PTS_PER_ARM; i++) {
            let t = i / (PTS_PER_ARM-1);

            // Parametric angle and radius for spiral
            let angle = lerp(armPhase, armPhase + spiralAngleSpread, t);
            let r = lerp(BASE_RADIUS, MAX_RADIUS, t);

            // Each arm's base color, modulated in time for shimmer
            let baseHue = (360/ARMS)*arm + 60*Math.sin(time*0.15 + arm);

            // Now add modulated waveform displacement
            let totalOffset = 0;

            // Overlay several waveforms with differing frequencies and speeds
            for (let w = 0; w < WAVE_COUNT; w++) {
                let freq = lerp(1, 5.8, w/(WAVE_COUNT-1));
                let speed = lerp(0.5, 2.5, w/(WAVE_COUNT-1));
                let amp = lerp(8, 34, Math.sin(w+time)*0.5+0.5) * (0.7 + 0.3*Math.sin(time*0.37+arm+w));
                totalOffset += Math.sin(angle * freq + time * speed + w*2 + arm) * amp / WAVE_COUNT;
            }

            let finalR = r + totalOffset;

            // Convert to canvas coordinates
            let x = centerX + finalR * Math.cos(angle);
            let y = centerY + finalR * Math.sin(angle);

            // Draw the path, modulate line width and alpha for glow
            if (i === 0)
                ctx.moveTo(x, y);
            else
                ctx.lineTo(x, y);

            // Optionally: draw small glowing points along path
            let pointHue = baseHue + 12*Math.sin(angle*2+time);
            let opacity = 0.22 + 0.28 * Math.sin(t*6 + time*1.8 + arm);
            ctx.save();
            ctx.globalAlpha = opacity + 0.18*Math.sin(i*0.32+arm+time*0.7);
            ctx.beginPath();
            ctx.arc(x, y, lerp(2.3, 6, Math.abs(Math.sin(angle*3+time*1.7-2*arm))),
                    0, 2*Math.PI);
            ctx.fillStyle = hsv2rgb(pointHue,0.77,1,1);
            ctx.shadowColor = hsv2rgb(pointHue,0.77,1,1);
            ctx.shadowBlur = 11;
            ctx.fill();
            ctx.restore();
        }
        // Draw the spiral arm as a continuous line with its own color
        ctx.save();
        ctx.lineWidth = lerp(2.1, 4.5, Math.sin(time+arm)*0.5+0.5);
        let strokeHue = ((360/ARMS)*arm + 246 + 60*Math.sin(time*0.26+arm*1.1))%360;
        ctx.globalAlpha = 0.44 + 0.14*Math.cos(time*1.18+arm);
        ctx.strokeStyle = hsv2rgb(strokeHue,0.62,1,1);
        ctx.shadowColor = hsv2rgb(strokeHue,0.92,1,1);
        ctx.shadowBlur = 14;
        ctx.stroke();
        ctx.restore();
    }

    // Draw central glow
    ctx.save();
    let pulse = 1.0 + 0.28 * Math.sin(time * 2.7);
    let glowRad = BASE_RADIUS * (1.08 + 0.11*Math.sin(time*1.3));
    let gradient = ctx.createRadialGradient(centerX,centerY,glowRad*0.15,
                                            centerX,centerY,glowRad*pulse);
    gradient.addColorStop(0, "rgba(255,245,224,0.22)");
    gradient.addColorStop(0.6, "rgba(160,211,255,0.17)");
    gradient.addColorStop(1, "rgba(0,0,0,0)");
    ctx.globalAlpha = 0.94;
    ctx.beginPath();
    ctx.arc(centerX, centerY, glowRad*pulse, 0, Math.PI*2);
    ctx.fillStyle = gradient;
    ctx.fill();
    ctx.restore();

    requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
</script>
```