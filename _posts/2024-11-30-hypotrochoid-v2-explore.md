---
layout: fullscreen
title: "Hypotrochoid Curve with Additional Scrollbars"
tags:
  - graphics
---

<style>
    canvas {
        display: block;
        margin: 0 auto;
        border: 1px solid black;
    }
    /* Desktop styles */
    @media screen and (min-width: 768px) {
        canvas {
            width: 800px;
            height: 800px;
        }
    }
    /* Mobile styles */
    @media screen and (max-width: 767px) {
        canvas {
            width: 90vw; /* Use 90% of viewport width */
            height: 90vw; /* Square canvas */
            max-width: 500px; /* Cap for larger mobile screens */
            max-height: 500px;
        }
    }

    .controls {
        margin-top: 20px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .control-group {
        margin: 10px 0;
        display: flex;
        align-items: center;
    }

    .control-group label {
        margin-right: 10px;
    }

    input[type="range"] {
        width: 200px;
    }

    .value-label {
        margin-left: 10px;
        font-weight: bold;
    }
</style>

<canvas id="canvas"></canvas>

<div class="controls">
    <div class="control-group">
        <label for="R">R (Outer radius):</label>
        <input type="range" id="R" min="50" max="300" value="240">
        <span id="R-value" class="value-label">240</span>
    </div>
    <div class="control-group">
        <label for="r">r (Inner radius):</label>
        <input type="range" id="r" min="10" max="150" value="58">
        <span id="r-value" class="value-label">58</span>
    </div>
    <div class="control-group">
        <label for="d">d (Distance):</label>
        <input type="range" id="d" min="10" max="150" value="150">
        <span id="d-value" class="value-label">150</span>
    </div>
    <div class="control-group">
        <label for="f1">f1 (Frequency 1):</label>
        <input type="range" id="f1" min="1" max="30" value="1">
        <span id="f1-value" class="value-label">1</span>
    </div>
    <div class="control-group">
        <label for="f2">f2 (Frequency 2):</label>
        <input type="range" id="f2" min="1" max="30" value="8">
        <span id="f2-value" class="value-label">8</span>
    </div>
    <div class="control-group">
        <label for="color">Select Color:</label>
        <input type="color" id="color" value="#0000ff">
    </div>
</div>

<script>
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    let R = 240;
    let r = 58;
    let d = 150;
    let selectedColor = '#0000ff';
    let f1 = 1;
    let f2 = 8;

    // Function to calculate the GCD
    function gcd(a, b) {
        return b ? gcd(b, a % b) : Math.abs(a);
    }

    // Set canvas dimensions based on CSS size and device pixel ratio
    function setCanvasSize() {
        const rect = canvas.getBoundingClientRect();
        const dpr = window.devicePixelRatio || 1;
        canvas.width = rect.width * dpr;
        canvas.height = rect.height * dpr;
        ctx.scale(dpr, dpr); // Adjust for high-DPI displays
    }

    function drawHypotrochoid() {
        setCanvasSize(); // Ensure canvas size is updated before drawing
        const width = canvas.getBoundingClientRect().width; // CSS width
        const height = canvas.getBoundingClientRect().height; // CSS height
        const centerX = width / 2;
        const centerY = height / 2;

        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.beginPath();
        ctx.strokeStyle = selectedColor;
        ctx.lineWidth = 2;

        const period = (2 * Math.PI * r) / gcd(R, r);
        for (let t = 0; t <= period; t += 0.01) {
            const x = centerX + (R - r) * Math.cos(f1 * t) * (1 + Math.cos(f2 * t)) + d * Math.cos(((R - r) / r) * t);
            const y = centerY + (R - r) * Math.sin(f1 * t) * (1 + Math.cos(f2 * t)) - d * Math.sin(((R - r) / r) * t);
            if (t === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
        }

        ctx.stroke();
    }

    // Event listeners for controls
    document.getElementById('R').addEventListener('input', function () {
        R = parseInt(this.value);
        document.getElementById('R-value').innerText = this.value;
        drawHypotrochoid();
    });

    document.getElementById('r').addEventListener('input', function () {
        r = parseInt(this.value);
        document.getElementById('r-value').innerText = this.value;
        drawHypotrochoid();
    });

    document.getElementById('d').addEventListener('input', function () {
        d = parseInt(this.value);
        document.getElementById('d-value').innerText = this.value;
        drawHypotrochoid();
    });

    document.getElementById('color').addEventListener('input', function () {
        selectedColor = this.value;
        drawHypotrochoid();
    });

    document.getElementById('f1').addEventListener('input', function () {
        f1 = parseInt(this.value);
        document.getElementById('f1-value').innerText = this.value;
        drawHypotrochoid();
    });

    document.getElementById('f2').addEventListener('input', function () {
        f2 = parseInt(this.value);
        document.getElementById('f2-value').innerText = this.value;
        drawHypotrochoid();
    });

    // Redraw on window resize
    window.addEventListener('resize', drawHypotrochoid);

    // Initial draw
    drawHypotrochoid();
</script>
