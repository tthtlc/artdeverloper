```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Psychedelic Evolving Art</title>
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
      overflow: hidden;
      background: #000;
    }
    body {
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100vh;
    }
    canvas {
      display: block;
      background: #000;
      box-shadow: 0 0 40px 20px #111;
    }
  </style>
</head>
<body>
  <canvas id="psychedelic"></canvas>
  <script>
    const canvas = document.getElementById('psychedelic');
    const ctx = canvas.getContext('2d');

    // Resize canvas to fit screen
    function resize() {
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    function hsvToRgb(h, s, v) {
      // h: 0-360, s: 0-1, v: 0-1
      let f = (n, k = (n + h/60) % 6) => v - v*s*Math.max(Math.min(k, 4-k, 1), 0);
      return [f(5)*255, f(3)*255, f(1)*255];
    }

    // Core animation function
    function drawPsychedelia(time) {
      const w = canvas.width;
      const h = canvas.height;
      const imgData = ctx.getImageData(0, 0, w, h);
      const d = imgData.data;

      // Animation parameters
      let t = time * 0.0004;
      let t2 = time * 0.0002;

      // Changes every frame for waving patterns
      for(let y = 0; y < h; y++) {
        for(let x = 0; x < w; x++) {
          // Calculate polar coordinates, normalized
          let nx = (x - w/2) / (0.5*Math.min(w,h));
          let ny = (y - h/2) / (0.5*Math.min(w,h));
          let r = Math.sqrt(nx*nx + ny*ny);
          let angle = Math.atan2(ny, nx);

          // Several overlapping colorwaves
          let wave1 = Math.sin(10*r - t*3 + Math.sin(angle*3 + t2));
          let wave2 = Math.sin(8*angle + t + Math.cos(r*7 + t2*2));
          let wave3 = Math.sin(8*r*r + t*2 - angle*5);

          // Blend the waves for "psychedelic" effect
          let combined = (wave1 + wave2 + wave3) / 3;

          // Calculate HSV with moving hue
          let hue = ((combined * 120) + (angle*180/Math.PI) + t*120) % 360;
          let sat = 0.8 + 0.2*Math.sin(wave3 + t2);
          let val = 0.8 + 0.2*Math.cos(wave2 + t);

          // Burst of rainbow in the center
          if (r < 0.3 + 0.1*Math.sin(t2*3 + r*10)) {
            hue = ((angle * 180 / Math.PI) + t*200) % 360;
            sat = 1.0;
            val = 1.0;
          }

          // Convert to RGB
          let [rr, gg, bb] = hsvToRgb(hue, Math.abs(sat), Math.abs(val));
          let idx = 4*(y*w + x);

          d[idx]   = rr;
          d[idx+1] = gg;
          d[idx+2] = bb;
          d[idx+3] = 245;
        }
      }
      ctx.putImageData(imgData, 0, 0);
    }

    // Animation loop
    function animate(time) {
      drawPsychedelia(time);
      requestAnimationFrame(animate);
    }

    animate(0);

    // Optional: click to make random flash
    canvas.addEventListener("click", ()=>{
      canvas.style.filter = "contrast(2) blur(2px)";
      setTimeout(()=>canvas.style.filter = "", 300);
    });
  </script>
</body>
</html>
```
