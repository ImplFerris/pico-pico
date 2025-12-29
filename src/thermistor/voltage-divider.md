# NTC and Voltage Divider

To understand how an NTC thermistor behaves in a voltage divider, I created a simple simulation using Falstad. In the circuit, the thermistor is shown as a resistor whose value changes with temperature.

You can import the circuit file I created, [voltage-divider-thermistor.circuitjs.txt](./voltage-divider-thermistor.circuitjs.txt), into the Falstad website and experiment with it yourself. I have also embedded a small simulator at the bottom of this page, so you can use either option.

In this configuration, the NTC thermistor is placed on the top of the voltage divider (R1), with a fixed resistor at the bottom (R2).  As the temperature changes, the resistance of the thermistor changes, and that directly affects the output voltage of the voltage divider.

> [!Important]
> Swapping the thermistor and the fixed resistor will invert the behavior of the output voltage.
> If you place the thermistor as R2 instead of R1, increase in temperature will cause \\( V_{out} \\) to decrease instead.

### Thermistor at 25°C

The thermistor has a resistance of 10kΩ at 25°C, resulting in an output voltage (\\( V_{out} \\)) of 1.65V.

<img style="display: block; margin: auto;" alt="pico2" src="./images/thermistor0.png"/>


## Thermistor at 10°C

The thermistor's resistance increases, resulting in a lower output voltage (\\( V_{out} \\)).

<img style="display: block; margin: auto;" alt="pico2" src="./images/thermistor2.png"/>

## Thermistor at 100°C

The thermistor's resistance decreases due to its negative temperature coefficient. This results in increase of output voltage.

<img style="display: block; margin: auto;" alt="pico2" src="./images/thermistor1.png"/>

## NTC Thermistor Voltage Divider Simulator

<style>
    .thermistor-sim-container {
        margin: 20px auto;
        max-width: 900px;
        padding: 20px;
        background: #282828;
        border-radius: 8px;
    }

    .thermistor-sim-title {
        text-align: center;
        color: #ebdbb2;
        margin-bottom: 20px;
        font-size: 20px;
        font-weight: bold;
    }

    .thermistor-sim-content {
        display: flex;
        gap: 20px;
        align-items: center;
        justify-content: center;
        flex-wrap: wrap;
    }

    .thermistor-sim-canvas {
        background: #1d2021;
        border-radius: 4px;
        border: 2px solid #3c3836;
    }

    .thermistor-sim-controls {
        background: #3c3836;
        padding: 20px;
        border-radius: 4px;
        min-width: 250px;
    }

    .thermistor-sim-label {
        display: block;
        margin-bottom: 10px;
        font-weight: bold;
        color: #ebdbb2;
        font-size: 14px;
    }

    .thermistor-sim-slider-wrap {
        display: flex;
        align-items: center;
        gap: 10px;
        margin-bottom: 15px;
    }

    .thermistor-sim-slider {
        flex: 1;
        height: 6px;
        border-radius: 3px;
        background: #504945;
        outline: none;
        -webkit-appearance: none;
    }

    .thermistor-sim-slider::-webkit-slider-thumb {
        -webkit-appearance: none;
        width: 16px;
        height: 16px;
        border-radius: 50%;
        background: #fe8019;
        cursor: pointer;
    }

    .thermistor-sim-slider::-moz-range-thumb {
        width: 16px;
        height: 16px;
        border-radius: 50%;
        background: #fe8019;
        cursor: pointer;
        border: none;
    }

    .thermistor-sim-value {
        min-width: 60px;
        text-align: right;
        font-weight: bold;
        color: #fe8019;
        font-size: 14px;
    }

    .thermistor-sim-indicator {
        width: 60px;
        height: 60px;
        border-radius: 50%;
        margin: 20px auto 0;
        transition: all 0.3s ease;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: bold;
        color: #1d2021;
        font-size: 24px;
    }
</style>

<div class="thermistor-sim-container">
    <div class="thermistor-sim-title">🌡️ NTC Thermistor Voltage Divider Simulator</div>
    <div class="thermistor-sim-content">
        <canvas id="thermistorSimCanvas" class="thermistor-sim-canvas" width="370" height="500"></canvas>
        <div class="thermistor-sim-controls">
            <label class="thermistor-sim-label">🌡️ Temperature</label>
            <div class="thermistor-sim-slider-wrap">
                <input type="range" class="thermistor-sim-slider" id="thermistorSimTemp" min="0" max="100" value="25">
                <span class="thermistor-sim-value" id="thermistorSimValue">25°C</span>
            </div>
            <div class="thermistor-sim-indicator" id="thermistorSimIndicator">25°</div>
        </div>
    </div>
</div>

<script>
    (function() {
        const canvas = document.getElementById('thermistorSimCanvas');
        const ctx = canvas.getContext('2d');
        const slider = document.getElementById('thermistorSimTemp');
        const valueDisplay = document.getElementById('thermistorSimValue');
        const indicator = document.getElementById('thermistorSimIndicator');
        
        const R2 = 10000; // Fixed resistor 10kΩ
        const Vin = 3.3;  // Supply voltage
        const R0 = 10000; // Resistance at 25°C
        const T0 = 25;    // Reference temperature
        const beta = 3950; // Beta coefficient (typical NTC)
        
        // Calculate thermistor resistance using simplified Beta equation
        function getThermistorResistance(tempC) {
            const T = tempC + 273.15; // Convert to Kelvin
            const T0K = T0 + 273.15;
            const R = R0 * Math.exp(beta * (1/T - 1/T0K));
            return R;
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
        
        function drawResistor(x, y, label, value, isThermistor) {
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
            
            // Draw thermistor symbol (diagonal line through resistor)
            if (isThermistor) {
                ctx.strokeStyle = '#fe8019';
                ctx.lineWidth = 2.5;
                ctx.setLineDash([]);
                ctx.beginPath();
                ctx.moveTo(x - 25, y - 25);
                ctx.lineTo(x + 25, y + 25);
                ctx.stroke();
                
                // Add "t°" symbol
                ctx.fillStyle = '#fe8019';
                ctx.font = 'bold 12px monospace';
                ctx.textAlign = 'center';
                ctx.fillText('t°', x - 35, y + 5);
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
            
            const temp = parseInt(slider.value);
            const R1 = getThermistorResistance(temp);
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
            drawResistor(rx, r1y, 'RT', r1Display, true);
            drawResistor(rx, r2y, 'R2', (R2/1000).toFixed(1) + 'kΩ', false);
            
            ctx.fillStyle = '#fabd2f';
            ctx.beginPath();
            ctx.arc(rx, my, 4, 0, Math.PI * 2);
            ctx.fill();
            
            drawVoltageLabel(rx, my, (Vout * 1000).toFixed(0) + ' mV');
        }
        
        function update() {
            const temp = parseInt(slider.value);
            valueDisplay.textContent = temp + '°C';
            
            // Color transition from cold (blue) to hot (red)
            let hue, lightness;
            if (temp < 50) {
                // 0-50°C: Blue to Yellow
                hue = 200 - (temp * 2.6); // 200 (blue) to 70 (yellow)
                lightness = 50 + (temp * 0.2);
            } else {
                // 50-100°C: Yellow to Red
                hue = 70 - ((temp - 50) * 1.2); // 70 (yellow) to 10 (red)
                lightness = 60 + ((temp - 50) * 0.2);
            }
            
            indicator.style.background = `radial-gradient(circle, hsl(${hue}, 80%, ${lightness}%), hsl(${hue}, 80%, ${lightness - 20}%))`;
            indicator.style.boxShadow = `0 0 ${temp * 0.4}px hsla(${hue}, 100%, 50%, ${temp / 150 + 0.2})`;
            indicator.textContent = temp + '°';
            
            drawCircuit();
        }
        
        slider.addEventListener('input', update);
        update();
    })();
</script>
