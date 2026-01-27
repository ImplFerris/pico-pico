# Simulation of LDR in Voltage Divider

To understand how an LDR behaves in a voltage divider, I created a simple simulation using Falstad. In the circuit, the LDR is shown as a resistor symbol with arrows pointing toward it, indicating incident light.

You can import the circuit file I created, [`voltage-divider-ldr.circuitjs.txt`](./voltage-divider-ldr.circuitjs.txt), import into the [Falstad site](https://www.falstad.com/circuit/e-voltdivide.html) and play around. I have also embedded a small simulator at the bottom of this page, so you can use either option.

In this configuration, the LDR is placed on the top of the voltage divider (i.e as R1), with a fixed resistor (as R2) at the bottom. This arrangement is commonly used because increasing light results in an increasing output voltage, which is intuitive when reading the value using an ADC.

Once the circuit is loaded, adjust the brightness slider and observe how the resistance of the LDR changes. At the same time, you can see how the output voltage \\( V_{out} \\) responds to those changes.

> [!Important]
> Swapping the LDR and the fixed resistor will invert the behavior of the output voltage.
> If you place the LDR as R2 (the bottom resistor) in the voltage divider, the relationship between light level and \\( V_{out} \\) is inverted because the equation changes.

### Example output for full brightness

When the LDR is exposed to strong light, its resistance is low. This causes a larger portion of the supply voltage to appear at the output, resulting in a higher

<img style="display: block; margin: auto;" alt="voltage-divider-ldr1" src="./images/voltage-divider-ldr1.png"/>

### Example output for low light

As the light level decreases, the resistance of the LDR increases. This reduces the output voltage.

<img style="display: block; margin: auto;" alt="voltage-divider-ldr2" src="./images/voltage-divider-ldr2.png"/>

### Example output for full darkness

In darkness, the LDR resistance becomes very high, causing the output voltage

<img style="display: block; margin: auto;" alt="voltage-divider-ldr3" src="./images/voltage-divider-ldr3.png"/>

## LDR Voltage Divider Simulator

<style>
.ldr-sim-container {
    margin: 20px 0;
    padding: 20px;
    background: #282828;
    border-radius: 8px;
    font-family: monospace;
}

.ldr-sim-title {
    text-align: center;
    color: #ebdbb2;
    margin-bottom: 20px;
    font-size: 20px;
    font-weight: bold;
}

.ldr-sim-content {
    display: flex;
    gap: 20px;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
}

.ldr-sim-canvas {
    background: #1d2021;
    border-radius: 4px;
    border: 2px solid #3c3836;
}

.ldr-sim-controls {
    background: #3c3836;
    padding: 20px;
    border-radius: 4px;
    min-width: 250px;
}

.ldr-sim-label {
    display: block;
    margin-bottom: 10px;
    font-weight: bold;
    color: #ebdbb2;
    font-size: 14px;
}

.ldr-sim-slider-wrap {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 15px;
}

.ldr-sim-slider {
    flex: 1;
    height: 6px;
    border-radius: 3px;
    background: #504945;
    outline: none;
    -webkit-appearance: none;
}

.ldr-sim-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #fabd2f;
    cursor: pointer;
}

.ldr-sim-slider::-moz-range-thumb {
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: #fabd2f;
    cursor: pointer;
    border: none;
}

.ldr-sim-value {
    min-width: 50px;
    text-align: right;
    font-weight: bold;
    color: #fabd2f;
    font-size: 14px;
}

.ldr-sim-indicator {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    margin: 0 auto;
    transition: all 0.3s ease;
}
</style>

<div class="ldr-sim-container">
    <div class="ldr-sim-title">⚡ LDR Voltage Divider Simulator</div>
<div class="ldr-sim-content">
    <canvas id="ldrSimCanvas" class="ldr-sim-canvas" width="350" height="500"></canvas>
    <div class="ldr-sim-controls">
    <label class="ldr-sim-label">💡 Light Brightness</label>
    <div class="ldr-sim-slider-wrap">
        <input type="range" class="ldr-sim-slider" id="ldrSimBrightness" min="0" max="100" value="50">
        <span class="ldr-sim-value" id="ldrSimValue">50%</span>
    </div>
    <div class="ldr-sim-indicator" id="ldrSimIndicator"></div>
    </div>
</div>
</div>

  <script>
    (function() {
      const canvas = document.getElementById('ldrSimCanvas');
      const ctx = canvas.getContext('2d');
      const slider = document.getElementById('ldrSimBrightness');
      const valueDisplay = document.getElementById('ldrSimValue');
      const indicator = document.getElementById('ldrSimIndicator');

      const R2 = 10000;
      const Vin = 3.3;

      function getLDRResistance(brightness) {
        const minR = 1000;
        const maxR = 1000000;
        const normalized = brightness / 100;
        return maxR * Math.pow(minR / maxR, normalized);
      }

      function drawWire(x1, y1, x2, y2) {
        ctx.strokeStyle = '#fabd2f';
        ctx.lineWidth = 2;
        ctx.setLineDash([6, 6]);
        ctx.beginPath();
        ctx.moveTo(x1, y1);
        ctx.lineTo(x2, y2);
        ctx.stroke();
      }

      function drawResistor(x, y, label, value, isLDR) {
        const h = 60, w = 15, z = 6;
        ctx.strokeStyle = '#fabd2f';
        ctx.lineWidth = 2;
        ctx.setLineDash([]);

        ctx.beginPath();
        ctx.moveTo(x, y - h/2);
        for (let i = 0; i < z; i++) {
          const yPos = y - h/2 + (i * h/z);
          ctx.lineTo(x - w/2, yPos + h/z/4);
          ctx.lineTo(x + w/2, yPos + 3*h/z/4);
        }
        ctx.lineTo(x, y + h/2);
        ctx.stroke();

        ctx.fillStyle = '#ebdbb2';
        ctx.font = 'bold 14px monospace';
        ctx.textAlign = 'left';
        ctx.fillText(label, x + 25, y - 5);
        ctx.font = '12px monospace';
        ctx.fillText(value, x + 25, y + 10);

        if (isLDR) {
          ctx.strokeStyle = '#fabd2f';
          ctx.fillStyle = '#fabd2f';
          ctx.lineWidth = 2;

          ctx.beginPath();
          ctx.moveTo(x - 35, y - 20);
          ctx.lineTo(x - 20, y - 10);
          ctx.stroke();
          ctx.beginPath();
          ctx.moveTo(x - 20, y - 10);
          ctx.lineTo(x - 20, y - 15);
          ctx.lineTo(x - 25, y - 10);
          ctx.fill();

          ctx.beginPath();
          ctx.moveTo(x - 35, y);
          ctx.lineTo(x - 20, y + 10);
          ctx.stroke();
          ctx.beginPath();
          ctx.moveTo(x - 20, y + 10);
          ctx.lineTo(x - 20, y + 5);
          ctx.lineTo(x - 25, y + 10);
          ctx.fill();
        }
      }

      function drawVoltageSource(x, y, voltage) {
        ctx.strokeStyle = '#fabd2f';
        ctx.lineWidth = 2;
        ctx.setLineDash([]);

        ctx.beginPath();
        ctx.moveTo(x - 20, y);
        ctx.lineTo(x + 20, y);
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x - 12, y + 12);
        ctx.lineTo(x + 12, y + 12);
        ctx.stroke();

        ctx.fillStyle = '#ebdbb2';
        ctx.font = '12px monospace';
        ctx.textAlign = 'right';
        ctx.fillText('Vin', x - 30, y + 5);
        ctx.fillText(voltage, x - 30, y + 18);
      }

      function drawVoltageLabel(x, y, voltage) {
        ctx.strokeStyle = '#fabd2f';
        ctx.lineWidth = 2;
        ctx.setLineDash([]);

        ctx.beginPath();
        ctx.moveTo(x, y);
        ctx.lineTo(x + 25, y);
        ctx.stroke();

        ctx.fillStyle = '#fe8019';
        ctx.font = 'bold 12px monospace';
        ctx.textAlign = 'left';
        ctx.fillText('Vout', x + 30, y - 2);
        ctx.fillText(voltage, x + 30, y + 12);
      }

      function drawCircuit() {
        ctx.fillStyle = '#1d2021';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        const brightness = parseInt(slider.value);
        const R1 = getLDRResistance(brightness);
        const Vout = Vin * (R2 / (R1 + R2));

        const lx = 80, rx = 270, ty = 80, r1y = 170, my = 260, r2y = 350, by = 440, vy = 250;

        drawWire(lx, ty, rx, ty);
        drawWire(rx, ty, rx, r1y - 30);
        drawWire(rx, r1y + 30, rx, my);
        drawWire(rx, my, rx, r2y - 30);
        drawWire(rx, r2y + 30, rx, by);
        drawWire(lx, by, rx, by);
        drawWire(lx, ty, lx, vy);
        drawWire(lx, vy + 10, lx, by);

        drawVoltageSource(lx, vy, Vin.toFixed(1) + 'V');

        let r1Display = R1 >= 1000 ? (R1/1000).toFixed(1) + 'kΩ' : R1.toFixed(0) + 'Ω';
        drawResistor(rx, r1y, 'R1', r1Display, true);
        drawResistor(rx, r2y, 'R2', (R2/1000).toFixed(1) + 'k');

        ctx.fillStyle = '#fabd2f';
        ctx.beginPath();
        ctx.arc(rx, my, 4, 0, Math.PI * 2);
        ctx.fill();

        drawVoltageLabel(rx, my, (Vout * 1000).toFixed(0) + ' mV');
      }

      function update() {
        const brightness = parseInt(slider.value);
        valueDisplay.textContent = brightness + '%';

        const lightness = 20 + (brightness * 0.5);
        indicator.style.background = `radial-gradient(circle, hsl(45, 70%, ${lightness}%), hsl(45, 70%, ${lightness - 15}%))`;
        indicator.style.boxShadow = `0 0 ${brightness * 0.3}px hsla(45, 100%, 50%, ${brightness / 100})`;

        drawCircuit();
      }

      slider.addEventListener('input', update);
      update();
    })();
</script>
