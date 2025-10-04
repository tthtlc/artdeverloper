---
layout: fullscreen
title: "Radiant Psychedelic Flower Petal Waves"
tags:
  - graphics
---

<style>
    body {
      background: radial-gradient(ellipse at center, #0a0026 0%, #111 100%);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
    }
    canvas {
      border: 2px solid #333;
      box-shadow: 0 0 24px #40176099;
      background: transparent;
    }
    .controls {
      display: flex;
      align-items: center;
      margin-top: 14px;
      font-family: monospace;
      color: #eee;
      user-select: none;
    }
    .label {
      margin-right: 8px;
      font-weight: bold;
      color: #e0caff;
      letter-spacing: 1px;
      font-size: 1.1em;
    }
    .slider-value {
      margin-left: 10px;
      color: #d2e6fa;
      min-width: 2em;
      text-align: right;
      display: inline-block;
    }
</style>
<canvas id="flowerCanvas" width="700" height="700"></canvas>
<div class="controls">
    <span class="label">Petals:</span>
    <input type="range" id="petalControl" min="4" max="18" value="8">
    <span id="petalValue" class="slider-value">8</span>
    <span class="label" style="margin-left:2em;">Waves:</span>
    <input type="range" id="waveControl" min="2" max="24" value="6">
    <span id="waveValue" class="slider-value">6</span>
</div>
<script>
    // Canvas/ctx
    const canvas = document.getElementById('flowerCanvas');
    const ctx = canvas.getContext('2d');

    // Controls
    const petalControl = document.getElementById('petalControl');
    const petalValue = document.getElementById('petalValue');
    const waveControl = document.getElementById('waveControl');
    const waveValue = document.getElementById('waveValue');

    // Animation state
    let PETALS = parseInt(petalControl.value, 10);       // Number of main petals
    let WAVES = parseInt(waveControl.value, 10);         // Number of sub-waves per petal
    let phi = 0;                                         // Global time phase

    // GUI: Dynamic labels
    petalValue.textContent = PETALS;
    waveValue.textContent = WAVES;

    petalControl.addEventListener('input',  (e) => {
      PETALS = parseInt(e.target.value, 10);
      petalValue.textContent = PETALS;
    });
    waveControl.addEventListener('input',  (e) => {
      WAVES = parseInt(e.target.value, 10);
      waveValue.textContent = WAVES;
    });

    // HSV to RGB util for luminous rainbows
    function hsv2rgb(h, s, v) {
      h %= 360; if (h < 0) h += 360;
      s /= 100; v /= 100;
      let c = v*s, x = c*(1-Math.abs((h/60)%2-1)), m = v-c;
      let [r,g,b]=[0,0,0];
      if(h<60)   [r,g,b]=[c,x,0];
      else if(h<120) [r,g,b]=[x,c,0];
      else if(h<180) [r,g,b]=[0,c,x];
      else if(h<240) [r,g,b]=[0,x,c];
      else if(h<300) [r,g,b]=[x,0,c];
      else           [r,g,b]=[c,0,x];
      return `rgb(${Math.round(255*(r+m))},${Math.round(255*(g+m))},${Math.round(255*(b+m))})`;
    }

    // Main animation
    function drawFlower(time) {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      const w = canvas.width, h = canvas.height;
      const cx = w/2, cy = h/2;
      const baseRadius = Math.min(w,h) * 0.22;
      const petalLength = baseRadius * 2.1;
      const innerR = baseRadius * 0.64;
      phi = time * 0.0007;

      // Layered faded background aura
      for(let bg=0;bg<5;bg++){
        ctx.save();
        const auraRadius = baseRadius*2.45 + 36*bg;
        ctx.globalAlpha = 0.11 - bg*0.018;
        ctx.beginPath();
        ctx.arc(cx, cy, auraRadius, 0, 2*Math.PI);
        ctx.closePath();
        ctx.fillStyle = hsv2rgb((300 + time*0.06 + 20*bg)%360, 63, 85-(13*bg));
        ctx.shadowColor = hsv2rgb((time*0.17 + 56 + 25*bg)%360,90, 96);
        ctx.shadowBlur = 64-bg*12;
        ctx.fill();
        ctx.restore();
      }

      // Draw petals
      ctx.save();
      ctx.translate(cx,cy);
      for(let p=0; p<PETALS; p++){
        // Each petal's phase and hue offset for movement/color cycling
        let theta = (p / PETALS) * 2*Math.PI;
        let rotation = theta + phi*0.53 + 1.3*Math.sin(phi*0.82);
        ctx.save();
        ctx.rotate(rotation);
        // Draw petal by plotting wave points
        ctx.beginPath();
        for(let i=0; i<=110; i++){
          let f = i/110;
          // Radial distance modulated by flower and wave harmonics
          let harmony = Math.sin(f*Math.PI) ** 0.62; // Petal profile
          let waviness =
            0.32*Math.sin(WAVES * f * Math.PI + phi*2 + p*2)
            + 0.12*Math.sin(2.2*WAVES * f * Math.PI + phi*3 + p*4)
            + 0.06*Math.sin(PETALS * f * Math.PI + phi*4.1 + p*1.11);

          let r = baseRadius + harmony*petalLength + harmony*waviness*baseRadius*1.2;
          let angle = (f-0.5)*Math.PI*0.98;

          // Animate subtle breathing by scaling slightly
          let tscale = 1 + 0.04*Math.sin(phi*1.7 + p);
          let x = Math.cos(angle)*r*tscale;
          let y = Math.sin(angle)*r*tscale;

          // Add roundedness at the tip via more harmony
          if(i===0) ctx.moveTo(x,y);
          else      ctx.lineTo(x,y);
        }
        ctx.closePath();
        // Color: Rainbow petals, hue animating over time
        let hue = (360*p/PETALS + 0.4*phi*360) % 360;
        let light = 68 + 18*Math.sin(phi + p);
        let sat   = 72 + 22*Math.cos(phi*0.88 + 0.9*p);
        ctx.fillStyle = hsv2rgb(hue, sat, light);
        ctx.shadowColor = hsv2rgb((hue+16)%360,sat+20,100);
        ctx.shadowBlur = 32;

        ctx.globalAlpha = 0.87;
        ctx.fill();

        // Glow stroke
        ctx.lineWidth = 2.8;
        ctx.strokeStyle = hsv2rgb((hue+33)%360, sat+14, 97);
        ctx.globalAlpha = 0.38;
        ctx.stroke();
        ctx.restore();
      }
      ctx.restore();

      // Inner pulsing nucleus
      let pulse = 0.56 + 0.23*Math.sin(phi*2.3) + 0.16*Math.cos(phi*3.7);
      ctx.save();
      ctx.globalAlpha = 0.82;
      ctx.beginPath();
      ctx.arc(cx, cy, innerR*pulse*1.06, 0, 2*Math.PI);
      ctx.closePath();
      ctx.fillStyle = hsv2rgb((time*0.13 + 100)%360, 80, 80);
      ctx.shadowBlur = 26;
      ctx.shadowColor = hsv2rgb((time*0.22 + 229)%360,85,95);
      ctx.fill();
      ctx.restore();

      // Inner core
      ctx.save();
      ctx.globalAlpha = 0.65;
      ctx.beginPath();
      ctx.arc(cx,cy, innerR*0.73*pulse, 0, 2*Math.PI);
      ctx.closePath();
      ctx.fillStyle = hsv2rgb((time*0.09+160)%360, 99, 97);
      ctx.shadowColor = "#fff6";
      ctx.shadowBlur = 18;
      ctx.strokeStyle = "#fff";
      ctx.lineWidth = 0.8+1.3*pulse;
      ctx.fill();
      ctx.stroke();
      ctx.restore();

      // Draw some radial "energy streaks"
      ctx.save();
      for(let s=0; s<16; s++){
        let ang = (s/16)*2*Math.PI + phi*0.91;
        let r1 = baseRadius*0.76 + 32*Math.sin(phi + s);
        let r2 = baseRadius*2.35 + 44*Math.cos(phi*1.43 + s*2);
        ctx.beginPath();
        ctx.moveTo(cx + Math.cos(ang)*r1, cy + Math.sin(ang)*r1);
        ctx.lineTo(cx + Math.cos(ang)*r2, cy + Math.sin(ang)*r2);
        let hue = (200 + s*20 + phi*222) % 360;
        ctx.strokeStyle = hsv2rgb(hue,88,88);
        ctx.lineWidth = 1.1 + 1.1*Math.sin(phi + s*2.5);
        ctx.globalAlpha = 0.10 + 0.10*Math.cos(time/1200 + s);
        ctx.stroke();
      }
      ctx.restore();
    }

    // Animation loop
    function animate(ts) {
      drawFlower(ts || 0);
      requestAnimationFrame(animate);
    }

    requestAnimationFrame(animate);
</script>
