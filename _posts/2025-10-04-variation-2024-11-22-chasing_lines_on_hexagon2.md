---
layout: fullscreen
title: "Psychedelic Radiating Waveform Dots"
tags:
  - graphics
---

<canvas id="canvas" width="900" height="700"></canvas>
<script>
(function() {
    const canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    const w = canvas.width;
    const h = canvas.height;

    // Parameters
    const NUM_ARMS = 12;
    const DOTS_PER_ARM = 90;
    const BASE_RADIUS = 90;
    const ARM_LENGTH = 290;
    const WAVE_FREQ = 3.3;
    const WAVE_SPEED = 0.020;
    const DOT_SIZE = 18;

    // For dynamic color cycling
    let globalHue = 180;
    let t = 0;

    // Draw a single psychedelic dot with glow
    function drawDot(x, y, r, hue, glow=1) {
        // Radial gradient for glow effect
        let grad = ctx.createRadialGradient(x, y, 0, x, y, r*2.2);
        grad.addColorStop(0, `hsla(${hue},90%,72%,1)`);
        grad.addColorStop(0.4, `hsla(${hue+30},100%,50%,0.55)`);
        grad.addColorStop(1, `hsla(${hue+60},100%,40%,0.0)`);

        ctx.save();
        ctx.globalCompositeOperation = "lighter";
        ctx.beginPath();
        ctx.arc(x, y, r*2.1, 0, Math.PI*2);
        ctx.fillStyle = grad;
        ctx.fill();
        ctx.restore();

        // Inner sharp dot
        ctx.save();
        ctx.shadowBlur = 22 * glow;
        ctx.shadowColor = `hsla(${hue},100%,70%,0.85)`;
        ctx.beginPath();
        ctx.arc(x, y, r, 0, Math.PI*2);
        ctx.fillStyle = `hsla(${hue},100%,70%,1)`;
        ctx.fill();
        ctx.restore();
    }

    function animate() {
        ctx.globalAlpha = 0.16;
        ctx.fillStyle = "#0c0017"; // slightly purple background, alpha fade trails
        ctx.fillRect(0, 0, w, h);
        ctx.globalAlpha = 1;

        let centerX = w/2, centerY = h/2;
        globalHue = (globalHue+0.6)%360;
        t += WAVE_SPEED;

        // For each "arm"
        for(let arm=0;arm<NUM_ARMS;arm++) {
            let theta = (arm/NUM_ARMS)*Math.PI*2;

            // The arm-phase shifts for swirling
            let armPhase = t*2.6 + arm*Math.PI/NUM_ARMS*1.5;

            for(let i=0;i<DOTS_PER_ARM;i++) {
                // Fraction along arm [0,1]
                let frac = i/(DOTS_PER_ARM-1);

                // Base radius out
                let baseRad = BASE_RADIUS + frac * ARM_LENGTH;

                // Main angular position
                let phi = theta + Math.sin(t*0.8+frac*5+arm)*0.09;

                // Wave offset (oscillates perpendicular to the arm)
                let wave = Math.sin(WAVE_FREQ*frac + armPhase) * 30
                         + Math.cos(WAVE_FREQ*(frac-0.25) + armPhase*1.07) * 14;

                // Radial offset, also pulsing in/out to 'breathe'
                let breathing = Math.sin(t*0.57+frac*4.0+arm)*16;

                // Final position
                let x = centerX + Math.cos(phi) * (baseRad + wave + breathing);
                let y = centerY + Math.sin(phi) * (baseRad + wave + breathing);

                // Color cycling: each arm is a different hue, waves of saturation/brightness
                let dotHue = (globalHue + arm*27 + 40*Math.sin(frac*8+armPhase*0.63))%360;
                let dotSize = DOT_SIZE * (0.44 + 0.61*Math.abs(Math.cos(armPhase+frac*6)));
                drawDot(x, y, dotSize, dotHue, 1.2 + 0.5 * Math.sin(frac*12 + t*2));
            }
        }
        // Optional: center pulsating core
        let coreRad = 32 + 12*Math.sin(t*2.3);
        drawDot(centerX, centerY, coreRad, (globalHue+180)%360, 2);

        requestAnimationFrame(animate);
    }

    // Responsive
    window.addEventListener('resize', ()=>{
      canvas.width=window.innerWidth;
      canvas.height=window.innerHeight;
    });

    // Start
    animate();
})();
</script>
