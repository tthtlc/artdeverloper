```markdown
---
layout: fullscreen
title: "Pulsing Mandala of Spiraling Waves and Neon Orbs"
tags:
  - graphics
---

<canvas id="mandalaCanvas" width="800" height="800" style="background:#110022;display:block;margin:0 auto;"></canvas>
<script>
const canvas = document.getElementById('mandalaCanvas');
const ctx = canvas.getContext('2d');
const w = canvas.width, h = canvas.height;
const centerX = w/2, centerY = h/2;

let t = 0;
const NUM_RINGS = 8;
const PETALS_PER_RING = 16;
const POINTS_PER_PETAL = 90;
const BASE_RADIUS = 80;

// Glow utility
function neonStroke(color, lineWidth, glow=16) {
    ctx.shadowColor = color;
    ctx.shadowBlur = glow;
    ctx.strokeStyle = color;
    ctx.lineWidth = lineWidth;
}

// Neon orb utility
function neonCircle(x, y, r, color, glow=20) {
    ctx.save();
    ctx.beginPath();
    ctx.arc(x, y, r, 0, Math.PI*2);
    ctx.shadowColor = color;
    ctx.shadowBlur = glow;
    ctx.globalAlpha = 0.93;
    ctx.fillStyle = color;
    ctx.fill();
    ctx.globalAlpha = 1;
    ctx.restore();
}

// Draw dynamic wave petal
function drawPetal(cx, cy, base_r, angle_offset, spread, phase, colorA, colorB) {
    ctx.save();
    ctx.beginPath();
    for(let i=0; i<=POINTS_PER_PETAL; i++) {
        // θ from 0 to spread
        let theta = angle_offset + (i/POINTS_PER_PETAL)*spread;
        // Pulsating radius, with extra higher freq component for detail
        let r = base_r * (1 + 0.22*Math.sin(3*theta + phase) + 0.09*Math.sin(12*theta + phase*1.4));
        // Petal breathing
        r *= 1 + 0.08 * Math.sin(phase + theta*7);

        let x = cx + Math.cos(theta) * r;
        let y = cy + Math.sin(theta) * r;
        if(i===0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
    }
    // Neon gradient along stroke
    let grad = ctx.createLinearGradient(
        cx + Math.cos(angle_offset)*base_r, cy + Math.sin(angle_offset)*base_r,
        cx + Math.cos(angle_offset+spread)*base_r, cy + Math.sin(angle_offset+spread)*base_r
    );
    grad.addColorStop(0, colorA);
    grad.addColorStop(1, colorB);
    neonStroke(grad, 2.8, 24);
    ctx.stroke();
    ctx.restore();
}

// Animation loop
function animate() {
    ctx.clearRect(0,0,w,h);
    t += 0.016; // Time in seconds

    // Mandala breathing
    let globalBreath = 0.10 + 0.20 * Math.sin(t*0.43);

    // Draw rings
    for(let ring=0; ring<NUM_RINGS; ring++) {
        // Expand outer rings
        let ringFrac = ring/(NUM_RINGS-1);
        let ringRadius = BASE_RADIUS + ring*55 + 30*Math.sin(t*0.7 + ring*0.8);

        // Spiral twist
        let twist = t*0.5 + ring*0.16;

        // Petal spread
        let spread = Math.PI*2/PETALS_PER_RING*0.9 + 0.2*Math.sin(t*0.4+ring);

        for(let p=0; p<PETALS_PER_RING; p++) {
            let baseAngle = (2*Math.PI*p/PETALS_PER_RING) + twist;

            // Unique color per petal by angle and ring
            let hue = (baseAngle*180/Math.PI + ring*30 + t*29)%360;
            let colorA = `hsl(${hue},98%,70%)`;
            let colorB = `hsl(${(hue+110)%360},100%,50%)`;

            // Petal phase animation
            let phase = t*0.9 + ring*0.5 + p*0.11;

            // Petal center
            let cx = centerX, cy = centerY;

            drawPetal(
                cx,
                cy,
                ringRadius * (1+globalBreath*ringFrac),
                baseAngle-spread/2, // symmetry
                spread,
                phase,
                colorA,
                colorB
            );

            // Draw trailing neon orbs along the spiral arms
            let orbN = 7 + 2*Math.sin(t + ring + p);
            for(let i=0; i<orbN; i++) {
                let frac = i/(orbN-1);
                let theta = baseAngle-spread/2 + frac*spread;
                let extraR = ringRadius * (1+globalBreath*ringFrac);
                extraR *= 1 + 0.13*Math.sin(t*1.1 + p*0.5 + frac*2.34 + ring)
                let orbRad = 5 + 3*Math.sin(t*1.12 + ring*0.19 + p*0.22 + i);
                let orbX = cx + Math.cos(theta)*extraR;
                let orbY = cy + Math.sin(theta)*extraR;

                let orbColor = `hsl(${(hue+frac*220)%360},97%,72%)`;
                neonCircle(orbX, orbY, orbRad, orbColor, 22 + 16*frac);
            }
        }
        // Accent: ring dots
        for(let p=0; p<PETALS_PER_RING; p++) {
            let dotA = (2*Math.PI*p/PETALS_PER_RING) + twist + 0.26 * Math.sin(t*0.9+ring);
            let rr = ringRadius * (1 + 0.17*Math.sin(t*0.7+ring));
            let dx = centerX + Math.cos(dotA)*rr;
            let dy = centerY + Math.sin(dotA)*rr;
            let hue = (dotA*180/Math.PI + ring*40 + t*46)%360;
            neonCircle(dx, dy, 3.7, `hsl(${hue},92%,82%)`);
        }
    }

    // Center pulse
    let pulseR = 28 + 6*Math.sin(t*2);
    neonCircle(centerX, centerY, pulseR, 'hsl('+(t*40%360)+',100%,85%)', 45);

    requestAnimationFrame(animate);
}
animate();
</script>
```
