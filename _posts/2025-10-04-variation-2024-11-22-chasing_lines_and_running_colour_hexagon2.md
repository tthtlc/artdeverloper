---
layout: fullscreen
title: "Psychedelic Wave Interference Garden"
tags:
  - graphics
---

<canvas id="canvas" width="900" height="700"></canvas>
<script>
/*
  Psychedelic Wave Interference Garden
  - By a creative generative artist, 2024
*/
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

const width = canvas.width;
const height = canvas.height;

// Parameters for waveforms and drifting "flowers"
const FLOWER_COUNT = 14;
const WAVE_LAYERS = 5;
const PETAL_COUNT = 16;
const BASE_RADIUS = 60;

let time = 0;

// Each flower has a center, phase, and color cycle
const flowers = [];
for (let i = 0; i < FLOWER_COUNT; i++) {
    const angle = (i / FLOWER_COUNT) * Math.PI * 2;
    const radius = 220 + 120 * Math.sin(i);
    flowers.push({
        cx: width/2 + radius * Math.cos(angle),
        cy: height/2 + radius * Math.sin(angle),
        baseAngle: angle + Math.random()*2,
        petalPhase: Math.random() * Math.PI * 2,
        colorOffset: Math.random() * 360,
        swirlSpeed: (Math.random()-0.5) * 0.0012 + 0.0011,
        orbitSpeed: (Math.random()-0.5) * 0.003 + 0.002
    });
}

function drawFlower(flowerIdx, t) {
    const flower = flowers[flowerIdx];

    // Animate flower's center orbiting the canvas
    const orbitR = 220 + 90 * Math.sin(flowerIdx*334 + t*flower.swirlSpeed*3);
    const angle = flower.baseAngle + t * flower.orbitSpeed;
    const cx = width/2 + orbitR * Math.cos(angle);
    const cy = height/2 + orbitR * Math.sin(angle);

    // Petal parameters with gentle animation
    for(let j=0; j<WAVE_LAYERS; j++) {
        let rotPhase = flower.petalPhase + t*0.06 + j*Math.PI/WAVE_LAYERS;
        const layerColor = (flower.colorOffset + j*38 + t*24) % 360;
        ctx.save();
        ctx.translate(cx, cy);

        ctx.beginPath();
        for(let k=0;k<=PETAL_COUNT;k++) {
            const theta = (k/PETAL_COUNT)*Math.PI*2;
            // Interference of radius: petals + modulating sine wave
            let petalWave = Math.sin(theta*3 + rotPhase) * 27 
                          + Math.sin(theta*5 - rotPhase*1.21) * 13
                          + Math.sin(t*0.021 + theta*8)*13;
            let r = BASE_RADIUS + j*19 + petalWave;

            // Iridescent pulsing of petals at edges
            if(j === WAVE_LAYERS-1) r += 23 * Math.sin(t*0.033+theta*13+flowerIdx);
            const x = r * Math.cos(theta);
            const y = r * Math.sin(theta);
            if(k===0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }
        ctx.closePath();
        // Fill with psychedelic radial gradient
        let grad = ctx.createRadialGradient(0,0,20+j*2,0,0,BASE_RADIUS+43+j*21);
        grad.addColorStop(0.0, `hsl(${(layerColor)%360},85%,70%)`);
        grad.addColorStop(0.4, `hsl(${(layerColor+80)%360},90%,54%)`);
        grad.addColorStop(0.85, `hsla(${(layerColor+180)%360},100%,45%,0.77)`);
        grad.addColorStop(1.0, `hsla(${(layerColor+260)%360},100%,60%,0)`);
        ctx.globalAlpha = 0.61 + 0.19*Math.sin(t*0.050+j*2+flowerIdx*4);
        ctx.fillStyle = grad;
        ctx.strokeStyle = `hsl(${(layerColor+32)%360},88%,65%)`;
        ctx.lineWidth = 2.7-j*0.3;
        ctx.fill();
        ctx.stroke();
        ctx.restore();
    }
}

function drawWaveBackground(t) {
    // Color cycling background with interfering transparent sine waves
    for(let w=0; w<4; w++) {
        ctx.save();
        ctx.globalAlpha = 0.14 + 0.06*Math.sin(w*65+t*0.01);
        const hue = (220 + w*43 + t*6.1)%360;
        ctx.strokeStyle = `hsl(${hue}, 81%, 63%)`;
        ctx.lineWidth = 4.6-w*0.7;
        ctx.beginPath();
        let freq = 1.5  + Math.sin(t*0.01+w*0.5)*0.6;
        let amp = 110 + w*32 + Math.sin(t*0.011+w)*24;
        for(let x=0; x<=width; x+=4) {
            let fx = (x/width)*Math.PI*2 * freq;
            let y = height/2 
                + amp * Math.sin(fx + t*0.023+w*3)
                + 32*Math.sin(fx*1.37 + t*0.019-w);
            ctx.lineTo(x, y);
        }
        ctx.stroke();
        ctx.restore();
    }
}

function animate() {
    time += 1;
    ctx.setTransform(1,0,0,1,0,0);
    // Subtle background fade for dreamy afterglows
    ctx.globalAlpha = 0.15;
    ctx.fillStyle = "#141a2c";
    ctx.fillRect(0,0,width,height);

    drawWaveBackground(time);

    // Central pulsating radiance
    ctx.save();
    ctx.globalAlpha = 0.18;
    let grad = ctx.createRadialGradient(width/2, height/2, 22, width/2, height/2, 400);
    grad.addColorStop(0.0,"hsl("+((220+Math.sin(time*0.01)*50)%360)+",60%,54%)");
    grad.addColorStop(1.0,"#000b  ");
    ctx.fillStyle = grad;
    ctx.fillRect(0,0,width,height);
    ctx.restore();

    // Paint all the flowers
    for (let i = 0; i < FLOWER_COUNT; i++) {
        drawFlower(i, time);
    }

    requestAnimationFrame(animate);
}

animate();


// Interactivity: Click to make all flowers "pulse" and swirl random petals
canvas.addEventListener("mousedown", function(e) {
    for(let i=0; i<FLOWER_COUNT; i++) {
        flowers[i].petalPhase += Math.random()*Math.PI*2;
        flowers[i].colorOffset += 40 + Math.random()*100;
    }
});
</script>
