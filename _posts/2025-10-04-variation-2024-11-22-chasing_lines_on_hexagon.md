---
layout: fullscreen
title: "Psychedelic Rippling Rings with Orbiting Particles"
tags:
  - graphics
---

<canvas id="canvas" width="800" height="600"></canvas>
<script>
function psychedelicRipplingRings() {
    const canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");
    const w = canvas.width;
    const h = canvas.height;

    const ringCount = 8;
    const particlesPerRing = 34;
    const ringBaseRadius = 60;
    const ringSpacing = 42;

    // Each ring has an evolving phase for rippling
    const ringPhases = Array.from({length: ringCount}, () => Math.random() * Math.PI * 2);

    // Generate initial particle phases for drifting on each ring
    const particlePhases = [];
    for (let i = 0; i < ringCount; i++) {
        particlePhases[i] = [];
        for (let j = 0; j < particlesPerRing; j++) {
            particlePhases[i][j] = Math.random() * Math.PI * 2;
        }
    }

    // For psychedelic color cycling, a global hue shift
    let globalHue = 0;

    // The parameters of the animation
    let time = 0;

    function draw() {
        ctx.globalCompositeOperation = 'source-over';
        // Faint semi-transparent fill for trails
        ctx.fillStyle = "rgba(8, 6, 21, 0.18)";
        ctx.fillRect(0, 0, w, h);

        // Shift the entire system slowly side to side
        const wobbleX = Math.sin(time * 0.22) * 16;
        const wobbleY = Math.cos(time * 0.18) * 12;

        for (let ring = 0; ring < ringCount; ring++) {
            // Ripple: Each ring radius pulses with a sine wave
            const phase = ringPhases[ring];
            const pulse = Math.sin(time * 0.23 + ring * 0.8 + phase) * (7 + ring * 0.5);
            const radius = ringBaseRadius + ring * ringSpacing + pulse;

            for (let p = 0; p < particlesPerRing; p++) {
                // Each particle orbits the center, with a phase offset and slow random drift
                const angle =
                    ((2 * Math.PI) / particlesPerRing) * p +
                    Math.sin(time * 0.11 + ring + particlePhases[ring][p]) * 0.24 +
                    time * (0.10 + ring * 0.004); // slow spin of the ring

                // Position with global wobble
                const x = w / 2 + Math.cos(angle) * radius + wobbleX;
                const y = h / 2 + Math.sin(angle) * radius + wobbleY;

                // The particle itself orbits a little within its own ring station
                const subAngle = angle * 1.4 + Math.sin(time * 0.8 + angle * 0.75) * 1.1;
                const ripple = Math.sin(time * 0.19 + angle + ring) * 6.8;

                const px = x + Math.cos(subAngle) * ripple;
                const py = y + Math.sin(subAngle) * ripple;

                // Psychedelic color: hue cycles with time, then varies per ring and per particle
                const hue = (globalHue + ring * 29 + p * 10 + Math.sin(time * 0.4 + ring) * 45) % 360;
                const sat = 70 + Math.sin(time * 0.18 + ring * 2) * 22;
                const lum = 64 + Math.cos(angle + time * 0.5) * 15;

                ctx.beginPath();
                // "Tails" - draw behind the particle for streaming effect
                for (let tail = 10; tail >= 1; tail -= 2) {
                    const tPhase = time - tail * 0.07;
                    const tAngle =
                        ((2 * Math.PI) / particlesPerRing) * p +
                        Math.sin(tPhase * 0.11 + ring + particlePhases[ring][p]) * 0.24 +
                        tPhase * (0.10 + ring * 0.004);

                    const tPulse = Math.sin(tPhase * 0.23 + ring * 0.8 + phase) * (7 + ring * 0.5);
                    const tRadius = ringBaseRadius + ring * ringSpacing + tPulse;

                    const tx = w / 2 + Math.cos(tAngle) * tRadius + wobbleX;
                    const ty = h / 2 + Math.sin(tAngle) * tRadius + wobbleY;
                    const tSubAngle = tAngle * 1.4 + Math.sin(tPhase * 0.8 + tAngle * 0.75) * 1.1;
                    const tRipple = Math.sin(tPhase * 0.19 + tAngle + ring) * 6.8;

                    const tailX = tx + Math.cos(tSubAngle) * tRipple;
                    const tailY = ty + Math.sin(tSubAngle) * tRipple;

                    ctx.globalAlpha = 0.25 * (tail/10);
                    ctx.arc(tailX, tailY, 5 - tail * 0.3, 0, Math.PI * 2);
                }
                // Main particle
                ctx.globalAlpha = 0.84;
                ctx.arc(px, py, 2.6 + Math.sin(time * 2 + angle*3 + ring*10) * 1.6, 0, Math.PI * 2);

                ctx.fillStyle = `hsl(${hue},${sat}%,${lum}%)`;
                ctx.shadowColor = `hsl(${(hue+34)%360},100%,${60+Math.sin(angle)*20}%)`;
                ctx.shadowBlur = 16 + Math.abs(Math.sin(time*0.5 + ring*2)) * 18;
                ctx.fill();
                ctx.shadowBlur = 0;
                ctx.globalAlpha = 1.0;
            }
        }

        // Draw core radiating rings beneath, subtle
        for (let r=0; r<ringCount; r++) {
            ctx.save();
            ctx.beginPath();
            const baseR = ringBaseRadius + r * ringSpacing +
                    Math.sin(time * 0.23 + r * 0.8 + ringPhases[r]) * (7 + r * 0.5);
            ctx.arc(w/2 + wobbleX, h/2 + wobbleY, baseR, 0, 2 * Math.PI);
            ctx.strokeStyle = `hsla(${globalHue + r * 25}, 50%, 80%, 0.09)`;
            ctx.lineWidth = 2.5 + Math.sin(time*0.9 + r) * 1.4;
            ctx.stroke();
            ctx.restore();
        }

        globalHue = (globalHue + 0.43) % 360;
        time += 0.0127;
        requestAnimationFrame(draw);
    }

    draw();
}
psychedelicRipplingRings();
</script>
