---
layout: fullscreen
title: Pulsing Psychedelic Hypnotunnel with Sine Rings
tags:
  - graphics
---

  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: radial-gradient(ellipse at center, #130022 0%, #000014 100%);
    }
    #controls {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(25, 14, 39, 0.7);
      padding: 10px 18px 8px 22px;
      border-radius: 14px;
      font-family: sans-serif;
      color: #eeecff;
      letter-spacing: 1.2px;
      z-index: 10;
      font-size: 1rem;
      box-shadow: 0 2px 14px #6664aa3f;
    }
    .slider {
      width: 120px;
    }
  </style>
  <canvas id="psychedelia"></canvas>
  <script>
    const canvas = document.getElementById('psychedelia');
    const ctx = canvas.getContext('2d');
    let width = window.innerWidth;
    let height = window.innerHeight;
    canvas.width = width;
    canvas.height = height;

    // Parameters, controlled by UI
    let tunnelDepth = 36;
    let pulseSpeed = 0.9;

    // Color palette
    function hsv2rgb(h, s, v) {
      let f = (n, k = (n + h / 60) % 6) =>
        v - v * s * Math.max(Math.min(k, 4 - k, 1), 0);
      return [f(5), f(3), f(1)];
    }

    // Handle controls
    document.getElementById('depth').addEventListener('input', (e) => {
      tunnelDepth = parseInt(e.target.value);
    });
    document.getElementById('speed').addEventListener('input', (e) => {
      pulseSpeed = parseFloat(e.target.value);
    });

    // Handle resizing
    function resize() {
      width = window.innerWidth;
      height = window.innerHeight;
      canvas.width = width;
      canvas.height = height;
    }
    window.addEventListener('resize', resize);

    // Core drawing loop
    let t = 0;
    function drawHypnotunnel() {
      ctx.clearRect(0, 0, width, height);

      const cx = width / 2;
      const cy = height / 2;
      const maxR = Math.min(width, height) * 0.47;

      // Draw rings deep into the tunnel
      for (let i = 0; i < tunnelDepth; i++) {
        let z = i / tunnelDepth;
        // Size of this ring, exaggerated near the observer
        let perspective = 1.4 / (z * 0.92 + 0.21);
        let r = maxR * Math.pow(z, 0.45) * perspective;
        // Sine pulse modulates the ring's thickness and wobbles its radius
        let wobble = Math.sin(t * pulseSpeed * (1.2 + 0.13*i) - i*0.758) * 0.11;
        let thickness = 16 + 14*Math.sin(t*0.77 + i*0.432) + 9*Math.cos(t*0.59 - i*0.655);
        let phase = t*1.035 + i*0.42 + wobble*4;
        let ringR = r * (0.96 + 0.07 * Math.sin(phase));
        // Color: cycle through psychedelic rainbow
        let h = (t*22 + i*17.8) % 360;
        let s = 0.56 + 0.22 * Math.sin(t*1.25 + i*0.19);
        let v = 0.93 - 0.21*z + 0.08*Math.cos(t*2.9 - i*1.1);
        let color = hsv2rgb(h, s, v);
        ctx.save();
        ctx.translate(cx, cy);

        // Draw ring by tracing a wavy path
        ctx.beginPath();
        let segments = Math.max(64, (128 + i * 4));
        for (let j = 0; j <= segments; j++) {
          let angle = (j/segments) * Math.PI * 2;
          // Apply sine-based "breathing" to the circumference
          let waviness = 1 +
            0.11 * Math.sin(8*angle - phase) +
            0.2 * Math.sin(3*angle + t - i*0.12);
          let px = Math.cos(angle) * ringR * waviness;
          let py = Math.sin(angle) * ringR * waviness;
          if (j === 0) ctx.moveTo(px, py);
          else ctx.lineTo(px, py);
        }
        ctx.closePath();
        ctx.lineWidth = Math.max(2, thickness * (1-z));
        ctx.strokeStyle = `rgb(${color[0]*255|0},${color[1]*255|0},${color[2]*255|0})`;
        ctx.shadowBlur = 24 + 60*z;
        ctx.shadowColor = `rgba(${color[0]*255|0},${color[1]*255|0},${color[2]*255|0},0.65)`;
        ctx.globalAlpha = 0.82 - 0.47*z;
        ctx.stroke();
        ctx.restore();
      }

      // Optionally draw a star field to enhance depth illusion
      let nStars = Math.floor(60 + (width*height) / 24000);
      ctx.save();
      ctx.globalAlpha = 0.13;
      ctx.fillStyle = "#fff";
      for (let s = 0; s < nStars; s++) {
        let rz = Math.random();
        let rr = maxR * Math.pow(0.74 + 0.35*rz, 1.1);
        let a = Math.random() * Math.PI * 2;
        let sx = cx + Math.cos(a) * rr * (1-0.13*rz);
        let sy = cy + Math.sin(a) * rr * (1-0.13*rz);
        ctx.beginPath();
        ctx.arc(sx, sy, 0.6 + rz*1.70, 0, Math.PI*2);
        ctx.fill();
      }
      ctx.restore();
    }

    // Animate
    function animate() {
      t += 0.0126 + (pulseSpeed-0.9)*0.011;
      drawHypnotunnel();
      requestAnimationFrame(animate);
    }
    animate();
</script>
