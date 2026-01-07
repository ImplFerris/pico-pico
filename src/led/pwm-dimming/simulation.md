# Simulation - LED Dimming with PWM

Here is a simulation to show the dimming effect on an LED based on the duty cycle and the High and Low parts of the square wave. I set the default speed very slow so it is clear and not annoying to watch. To start it, click the “Start animation” button. You can increase the speed by reducing the delay time and watching the changes.

<style>
.pwm-container {
    text-align: center;
    padding: 20px;
    max-width: 900px;
    margin: 0 auto;
}

.pwm-title {
    margin-bottom: 5px;
    font-size: 24px;
    font-weight: 500;
}

.pwm-subtitle {
    color: #888;
    margin-bottom: 20px;
    font-size: 14px;
}

.pwm-led-section {
    background: rgba(255, 255, 255, 0.03);
    border-radius: 15px;
    padding: 30px;
    margin-bottom: 20px;
    border: 1px solid rgba(255, 255, 255, 0.05);
}

.pwm-led-container {
    position: relative;
    margin: 20px auto;
    width: 180px;
    height: 180px;
}

.pwm-led {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background: radial-gradient(circle at 35% 35%, #ff5555, #dd0000);
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    border: 3px solid #440000;
}

.pwm-led-glow {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 160px;
    height: 160px;
    border-radius: 50%;
    background: radial-gradient(circle, rgba(255, 68, 68, 0.5), transparent 70%);
    filter: blur(30px);
}

.pwm-duty-info {
    margin-top: 20px;
}

.pwm-duty-cycle {
    font-size: 48px;
    font-weight: 600;
    color: #ff5555;
    margin: 10px 0;
    text-shadow: 0 0 15px rgba(255, 85, 85, 0.4);
}

.pwm-duty-label {
    color: #999;
    font-size: 13px;
    text-transform: uppercase;
    letter-spacing: 1px;
}

.pwm-phase {
    color: #888;
    font-size: 15px;
    margin-top: 10px;
}

.pwm-waveform-section {
    background: rgba(0, 0, 0, 0.2);
    border-radius: 15px;
    padding: 20px;
    margin-bottom: 20px;
    border: 1px solid rgba(255, 255, 255, 0.05);
}

.pwm-waveform-title {
    font-size: 15px;
    margin-bottom: 15px;
    color: #aaa;
    font-weight: 500;
}

#pwm-waveform {
    background: #0a0a15;
    border-radius: 8px;
    display: block;
    margin: 0 auto;
    border: 1px solid #222;
    max-width: 100%;
}

.pwm-controls-section {
    background: rgba(255, 255, 255, 0.03);
    border-radius: 15px;
    padding: 20px;
    border: 1px solid rgba(255, 255, 255, 0.05);
}

.pwm-speed-control {
    margin-bottom: 20px;
}

.pwm-speed-label {
    color: #aaa;
    font-size: 14px;
    margin-bottom: 8px;
}

.pwm-speed-value {
    color: #ff5555;
    font-weight: 600;
    font-size: 16px;
    margin-bottom: 10px;
}

#pwm-speedSlider {
    width: 100%;
    max-width: 300px;
    height: 6px;
    border-radius: 3px;
    background: #333;
    outline: none;
    -webkit-appearance: none;
}

#pwm-speedSlider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: #ff5555;
    cursor: pointer;
    box-shadow: 0 0 10px rgba(255, 85, 85, 0.5);
}

#pwm-speedSlider::-moz-range-thumb {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: #ff5555;
    cursor: pointer;
    border: none;
    box-shadow: 0 0 10px rgba(255, 85, 85, 0.5);
}

.pwm-buttons {
    margin-top: 15px;
}

.pwm-buttons button {
    background: linear-gradient(135deg, #ff5555 0%, #cc0000 100%);
    color: white;
    border: none;
    padding: 12px 35px;
    font-size: 15px;
    border-radius: 8px;
    cursor: pointer;
    margin: 0 8px;
    font-weight: 500;
    transition: all 0.2s;
    box-shadow: 0 3px 12px rgba(255, 85, 85, 0.25);
}

.pwm-buttons button:hover:not(:disabled) {
    transform: translateY(-1px);
    box-shadow: 0 5px 15px rgba(255, 85, 85, 0.35);
}

.pwm-buttons button:active:not(:disabled) {
    transform: translateY(0);
}

.pwm-buttons button:disabled {
    background: #333;
    cursor: not-allowed;
    box-shadow: none;
    opacity: 0.5;
}
</style>

<div class="pwm-container">
    <div class="pwm-led-section">
        <div class="pwm-led-container">
            <div class="pwm-led-glow" id="pwm-glow"></div>
            <div class="pwm-led" id="pwm-led"></div>
        </div>
        <div class="pwm-duty-info">
            <div class="pwm-duty-label">Duty Cycle</div>
            <div class="pwm-duty-cycle" id="pwm-dutyCycle">0%</div>
            <div class="pwm-phase" id="pwm-phase">Ready to start</div>
        </div>
    </div>
    <div class="pwm-waveform-section">
        <div class="pwm-waveform-title">PWM Square Wave Signal</div>
        <canvas id="pwm-waveform" width="800" height="150"></canvas>
    </div>
    <div class="pwm-controls-section">
        <div class="pwm-speed-control">
            <div class="pwm-speed-label">Animation Speed</div>
            <div class="pwm-speed-value" id="pwm-speedValue">Medium (50ms)</div>
            <input type="range" id="pwm-speedSlider" min="20" max="200" value="200" step="10">
        </div>
        <div class="pwm-buttons">
            <button id="pwm-startBtn">Start Animation</button>
            <button id="pwm-stopBtn" disabled>Stop</button>
        </div>
    </div>
</div>

<script>
    (function() {
    const led = document.getElementById('pwm-led');
    const glow = document.getElementById('pwm-glow');
    const dutyCycleDisplay = document.getElementById('pwm-dutyCycle');
    const phaseDisplay = document.getElementById('pwm-phase');
    const startBtn = document.getElementById('pwm-startBtn');
    const stopBtn = document.getElementById('pwm-stopBtn');
    const speedSlider = document.getElementById('pwm-speedSlider');
    const speedValue = document.getElementById('pwm-speedValue');
    const canvas = document.getElementById('pwm-waveform');
    const ctx = canvas.getContext('2d');

    let animationRunning = false;
    let currentDutyCycle = 0;
    let waveformOffset = 0;
    let animationSpeed = 50;

    function updateSpeedDisplay() {
        const speed = parseInt(speedSlider.value);
        let label;
        if (speed <= 40) label = 'Very Fast';
        else if (speed <= 70) label = 'Fast';
        else if (speed <= 100) label = 'Medium';
        else if (speed <= 150) label = 'Slow';
        else label = 'Very Slow';
        speedValue.textContent = `${label} (${speed}ms)`;
        animationSpeed = speed;
    }

    speedSlider.addEventListener('input', updateSpeedDisplay);

    function setDutyCycle(percent) {
        currentDutyCycle = percent;
        const brightness = percent / 100;
        led.style.opacity = brightness;
        glow.style.opacity = brightness;
        dutyCycleDisplay.textContent = `${percent}%`;
    }

    function drawWaveform() {
        const width = canvas.width;
        const height = canvas.height;
        const period = 160;
        const highVoltage = 30;
        const lowVoltage = height - 30;

        ctx.fillStyle = '#0a0a15';
        ctx.fillRect(0, 0, width, height);

        // Draw subtle grid
        ctx.strokeStyle = '#1a1a2a';
        ctx.lineWidth = 1;
        for (let x = 0; x < width; x += 40) {
            ctx.beginPath();
            ctx.moveTo(x, 0);
            ctx.lineTo(x, height);
            ctx.stroke();
        }
        for (let y = 0; y < height; y += 30) {
            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(width, y);
            ctx.stroke();
        }

        // Draw voltage labels
        ctx.fillStyle = '#555';
        ctx.font = '11px monospace';
        ctx.fillText('HIGH', 8, highVoltage + 15);
        ctx.fillText('LOW', 8, lowVoltage + 5);

        // Draw duty cycle percentage on waveform
        ctx.fillStyle = '#ff5555';
        ctx.font = 'bold 14px sans-serif';
        ctx.fillText(`${currentDutyCycle}%`, width - 60, 25);

        // Draw PWM waveform
        ctx.strokeStyle = '#ff5555';
        ctx.lineWidth = 2.5;
        ctx.beginPath();

        const onTime = (currentDutyCycle / 100) * period;
        let x = -waveformOffset;
        let isHigh = true;

        while (x < width) {
            if (isHigh) {
                ctx.lineTo(x, lowVoltage);
                ctx.lineTo(x, highVoltage);
                ctx.lineTo(x + onTime, highVoltage);
                ctx.lineTo(x + onTime, lowVoltage);
                x += onTime;
                isHigh = false;
            } else {
                const offTime = period - onTime;
                ctx.lineTo(x + offTime, lowVoltage);
                x += offTime;
                isHigh = true;
            }
        }

        ctx.stroke();

        waveformOffset = (waveformOffset + 1) % period;
    }

    function animate() {
        if (animationRunning) {
            drawWaveform();
            requestAnimationFrame(animate);
        }
    }

    async function sleep(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }

    async function pwmLoop() {
        while (animationRunning) {
            phaseDisplay.textContent = 'Fading In...';
            for (let i = 0; i <= 100 && animationRunning; i++) {
                setDutyCycle(i);
                await sleep(animationSpeed);
            }

            if (!animationRunning) break;

            phaseDisplay.textContent = 'Fading Out...';
            for (let i = 100; i >= 0 && animationRunning; i--) {
                setDutyCycle(i);
                await sleep(animationSpeed);
            }

            if (!animationRunning) break;

            phaseDisplay.textContent = 'Paused (1500ms)';
            await sleep(1500);
        }
    }

    startBtn.addEventListener('click', () => {
        if (!animationRunning) {
            animationRunning = true;
            startBtn.disabled = true;
            stopBtn.disabled = false;
            speedSlider.disabled = true;
            animate();
            pwmLoop();
        }
    });

    stopBtn.addEventListener('click', () => {
        animationRunning = false;
        startBtn.disabled = false;
        stopBtn.disabled = true;
        speedSlider.disabled = false;
        phaseDisplay.textContent = 'Stopped';
    });

    updateSpeedDisplay();
    setDutyCycle(0);
    drawWaveform();
})();
</script>
