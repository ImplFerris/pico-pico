
# Pulse Width Modulation (PWM)

<style>
  
  .slider-container {
    margin: 20px 0;
  }

  label {
    margin-right: 10px;
  }

  .led-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2px;
    position: relative;
  }

  .led-body {
    width: 30px;
    height: 40px;
    background: radial-gradient(circle at center, #ff5555, #cc0000);
    border-radius: 50% 50% 0 0;
    border: 2px solid #990000;
    position: relative;
    box-shadow: 0 0 10px rgba(255, 85, 85, 0.8);
  }

  .led-body::after {
    content: '';
    position: absolute;
    top: 5px;
    left: 7px;
    width: 16px;
    height: 16px;
    background: rgba(255, 255, 255, 0.4);
    border-radius: 50%;
  }

  .led-pin {
    width: 2px;
    height: 40px;
    background-color: #333;
    position: relative;
  }

  .anode {
    height: 50px; /* Longer pin for the anode */
    margin-right: 15px;
    background-color: #666;
  }

  .cathode {
    height: 40px; /* Shorter pin for the cathode */
    margin-left: 15px;
    background-color: #333;
    position: absolute;
    margin-top: 45px;
  }

  canvas {
    border: 1px solid #ccc;
    display: block;
    margin: 10px auto;
  }
  #pwmCanvas {
    background-color: #fefefe;
  } 
</style>

PWM stands for Pulse Width Modulation. It's a technique that uses a digital signal switching rapidly between HIGH and LOW to simulate an analog output.

Instead of providing a steady voltage like 1.5V as a true analog signal would, PWM quickly toggles between full voltage and 0V. By controlling how long the signal stays HIGH versus LOW, you can control the average power or voltage delivered to the device.

### How PWM Works

PWM works by sending a square wave signal that switches between HIGH and LOW at a fixed frequency. The key parameter is the "duty cycle", which is the percentage of time the signal is HIGH during one complete cycle.

<img style="display: block; margin: auto;" alt="LED PWM" src="../images/led-pwm.jpg" />

For example:
- A 0% duty cycle means the signal is always LOW (0V average). 
- A 50% duty cycle means the signal is HIGH and LOW for equal amounts of time (1.65V average on a 3.3 system). 
- A 75% duty cycle means the signal is HIGH for 75% of the time and LOW for 25% of the time. 
- A 100% duty cycle means the signal is always HIGH (3.3V).

The switching happens extremely fast, typically hundreds or thousands of times per second. This is so fast that devices like LEDs and motors respond to the average voltage rather than seeing individual pulses.

## Example Usage: Dimming an LED

An LED flashes so quickly that your eyes can't see individual ON and OFF pulses, so you perceive only the average brightness. A low duty cycle makes it look dim, a higher one makes it look brighter, even though the LED is always switching between full voltage and zero.

## Example Usage: Controlling a Servo Motor

A servo reads the width of a pulse to decide its angle. It expects a pulse every 20 milliseconds, and the pulse width - about 1ms for 0°, 1.5ms for 90°, and 2ms for 180° - tells the servo where to move.


## Period and Frequency

In PWM, period and frequency work together to determine how fast the signal switches between HIGH and LOW states, which directly affects how smooth and effective your control is.

Period is the total time for one on-off cycle to complete. 

<img style="display: block; margin: auto;" alt="Period and Frequency" src="./images/pwm-period-frequency.svg"/>

The frequency is the number of cycles it completes in one second, measured in Hertz (Hz).  Frequency is the inverse of the period.  So, a higher frequency means a shorter period, resulting in faster switching between HIGH and LOW states.

\\[
\text{Frequency (Hz)} = \\frac{1}{\text{Period (s)}}
\\]

So if the period is 1 second, then the frequency will be 1HZ.

\\[
1 \text{Hz} = \\frac{1 \text{ cycle}}{1 \text{ second}} = \\frac{1}{1 \text{ s}}
\\]

For example, if the period is 20ms(0.02s), the frequency will be 50Hz.

\\[
\text{Frequency} = \\frac{1}{20 \text{ ms}} = \\frac{1}{0.02 \text{ s}} = 50 \text{ Hz}
\\]


**Calculating Cycle count from Frequency per second**

The Formula to calculate cycle count:  
\\[
\text{Cycle Count} = \text{Frequency (Hz)} \\times \text{Total Time (seconds)}
\\]

If a PWM signal has a frequency of 50Hz, it means it completes 50 cycles in one second.


## Simulation

Here is the interactive simulation. Use the sliders to adjust the duty cycle and frequency, and watch how the pulse width and LED brightness change. The upper part of the square wave represents when the signal is high (on). The lower part represents when the signal is low (off). The width of the high portion changes with the duty cycle.

<canvas id="pwmCanvas" width="800" height="200"></canvas>
<div class="led-container">
  <div class="led-body" id="ledBody"></div>
  <div class="led-pin anode"></div>
  <div class="led-pin cathode"></div>
</div>

<div class="slider-container">
  <label for="dutyCycle">Duty Cycle (%): </label>
  <input type="range" id="dutyCycle" min="0" max="100" value="50">
  <span id="dutyCycleValue">50</span>%
</div>
<div class="slider-container">
  <label for="frequency">Frequency (Hz): </label>
  <input type="range" id="frequency" min="1" max="50" value="10">
  <!-- <span id="frequencyValue">x</span> Hz -->
</div>

If you change the duty cycle from "low to high" and "high to low" in the simulation, you should notice the LED kind of giving a dimming effect.


<script>
  const pwmCanvas = document.getElementById('pwmCanvas');
  const pwmCtx = pwmCanvas.getContext('2d');
  
  const dutyCycleSlider = document.getElementById('dutyCycle');
  const dutyCycleValue = document.getElementById('dutyCycleValue');
  const frequencySlider = document.getElementById('frequency');
  const frequencyValue = document.getElementById('frequencyValue');
  const ledBody = document.getElementById('ledBody');

  let dutyCycle = 50; // Initial duty cycle in percentage
  let frequency = 10; // Initial frequency in Hz

  function drawPWM() {
    pwmCtx.clearRect(0, 0, pwmCanvas.width, pwmCanvas.height);

    const period = 1000 / frequency; // Period in ms
    const onTime = period * (dutyCycle / 100); // On time in ms
    const offTime = period - onTime; // Off time in ms

    const totalWidth = pwmCanvas.width;
    const cycles = frequency; // Number of cycles to display
    const cycleWidth = totalWidth / cycles;

    pwmCtx.strokeStyle = 'black';
    pwmCtx.lineWidth = 2;
    pwmCtx.beginPath();

    let x = 0;

    if (dutyCycle === 100) {
      pwmCtx.moveTo(0, 50);
      pwmCtx.lineTo(pwmCanvas.width, 50);
    } else if (dutyCycle === 0) {
      pwmCtx.moveTo(0, 150);
      pwmCtx.lineTo(pwmCanvas.width, 150);
    } else {
      for (let i = 0; i < cycles; i++) {
        const highWidth = (onTime / period) * cycleWidth;
        const lowWidth = (offTime / period) * cycleWidth;

        pwmCtx.moveTo(x, 50);
        pwmCtx.lineTo(x + highWidth, 50);
        pwmCtx.lineTo(x + highWidth, 150);
        pwmCtx.lineTo(x + highWidth + lowWidth, 150);
        pwmCtx.lineTo(x + highWidth + lowWidth, 50);

        x += cycleWidth;
      }
    }
    pwmCtx.stroke();
  }

  function updateLED() {
    const brightness = dutyCycle / 100;
    
    ledBody.style.background = `radial-gradient(circle at center, rgba(255, 85, 85, ${brightness}), #cc0000)`;
  }

  function update() {
    dutyCycle = parseInt(dutyCycleSlider.value, 10);
    frequency = parseInt(frequencySlider.value, 10);

    dutyCycleValue.textContent = dutyCycle;
    // frequencyValue.textContent = frequency;

    drawPWM();
    updateLED();
  }

  dutyCycleSlider.addEventListener('input', update);
  frequencySlider.addEventListener('input', update);

  // Initial draw
  drawPWM();
  updateLED();
</script>
