# Blinking an External LED

From now on, we'll use more external parts with the Pico. Before we get there, it helps to get comfortable with simple circuits and how to connect components to the Pico's pins. In this chapter, we'll start with something basic: blinking an LED that's connected outside the board.

## Hardware Requirements

- LED
- Resistor
- Jumper wires

## Components Overview

1. LED: An LED (Light Emitting Diode) lights up when current flows through it. The longer leg (anode) connects to positive, and the shorter leg (cathode) connects to ground. We'll connect the anode to GP13 (with a resistor) and the cathode to GND.

2. Resistors: A resistor limits the current in a circuit to protect components like LEDs. Its value is measured in Ohms (Ω). We'll use a 330 ohm resistor to safely power the LED.

<table>
  <thead>
    <tr>
      <th>Pico Pin</th>
      <th style="width: 250px; margin: 0 auto;">Wire</th>
      <th>Component</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 13</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="female-left"></div>
          <div class="female-right"></div>
        </div>
      </td>
      <td>Resistor</td>
    </tr>
    <tr>
      <td>Resistor</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="female-left"></div>
          <div class="female-right"></div>
        </div>
      </td>
      <td>Anode (long leg) of LED</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="female-left"></div>
          <div class="female-right"></div>
        </div>
      </td>
      <td>Cathode (short leg) of LED</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;" alt="pico2" src="../images/pico-external-led.png"/>

You can connect the Pico to the LED using jumper wires directly, or you can place everything on a breadboard. If you're unsure about the hardware setup, you can also refer the [Raspberry pi guide](https://projects.raspberrypi.org/en/projects/introduction-to-the-pico/7).

<div class="image-with-caption" style="text-align:center; ">
    <img src="./images/pico-2-rp2350-with-external-led.png" alt="Connecting External LED with Pico 2 (RP2350)" style="max-width:100%; height:auto; display:block; margin:0 auto;"/>
    <div class="caption" style="font-size:0.9em; color:#555; margin-top:6px;">Circuit with Breadboard</div>
</div>

> [!TIP]
> On the Pico, the pin labels are on the back of the board, which can feel inconvenient when plugging in wires. I often had to check the pinout diagram whenever I wanted to use a GPIO pin. Use the Raspberry Pi logo on the front as a reference point and match it with the [pinout diagram](../pico2-pinout.md) to find the correct pins. Pin positions 2 and 39 are also printed on the front and can serve as additional guides.

## LED Blink - Simulation

<style>
.wrap{
  margin:0px auto;
  padding:18px;
  background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  border-radius:10px;
  box-shadow:0 8px 24px rgba(2,6,23,0.45);
  color:var(--mm-text, #e6eef8);
  font-family:ui-monospace,SFMono-Regular,Menlo,Monaco,monospace;
}

.top{display:flex;gap:18px;align-items:center;flex-wrap:wrap}

/* LED visual */
.led{
  width:56px;height:56px;border-radius:50%;background:#334155;display:flex;align-items:center;justify-content:center;
  box-shadow:0 0 0 6px rgba(16,185,129,0.02), inset 0 -6px 12px rgba(0,0,0,0.5);
  transition:background 180ms,box-shadow 180ms,transform 180ms;
}
.led.on{
  background:radial-gradient(circle at 35% 30%, #bbf7d0, #10b981);
  box-shadow:0 0 18px rgba(16,185,129,0.45), 0 6px 20px rgba(0,0,0,0.45);
  transform:scale(1.06);
}
.led-label{font-size:13px;color:#94a3b8;margin-top:8px;text-align:center}

/* Code lines */
.code{
  margin-top:8px;
  background:#0b1220;
  padding:12px;
  border-radius:8px;
  font-size:14px;
  color:#cbd5e1;
  overflow:auto;
}
.line{padding:6px 8px;border-radius:6px;display:flex;align-items:center;gap:10px}
.line .num{width:28px;color:#94a3b8;text-align:right;padding-right:6px}
.line .text{white-space:pre}
.line.current{
  background:linear-gradient(90deg, rgba(255,0,0,0.35), rgba(255,150,150,0.25));
  box-shadow:inset 0 0 10px rgba(255,0,0,0.45);
}

/* progress bar */
.progress-wrap{margin-top:10px}
.progress{height:12px;background:rgba(255,255,255,0.04);border-radius:8px;overflow:hidden}
.bar{height:100%;width:0%;background:linear-gradient(90deg, rgba(16,185,129,0.95), rgba(34,197,94,0.95));transition:width 0.06s linear}
.progress-info{display:flex;justify-content:space-between;margin-top:6px;color:#94a3b8;font-size:13px}

/* controls */
.controls{margin-top:12px;display:flex;gap:10px;align-items:center;flex-wrap:wrap}
button{background:#0b1826;border:1px solid rgba(255,255,255,0.03);color:inherit;padding:8px 10px;border-radius:8px;cursor:pointer}
button:active{transform:translateY(1px)}
input[type=number]{width:110px;padding:8px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:#071023;color:inherit}
</style>

In this simulation I set the default delay to 5000 milliseconds so the animation is calmer and easier to follow. You can lower it to something like 500 milliseconds to see the LED blink more quickly. When we run the actual code on the Pico, we will use a 500 millisecond delay.

<div class="wrap">
  <div class="top">
    <div style="display:flex;flex-direction:column;align-items:center">
      <div id="led" class="led" aria-hidden="true"></div>
      <div class="led-label"><strong id="led-state">LOW</strong></div>
    </div>
    <div style="flex:1">
      <div class="code" id="code">
        <div class="line" data-index="0"><div class="num">1</div><div class="text">let mut led = Output::new(p.PIN_13, Level::Low);</div></div>
        <div class="line" data-index="1"><div class="num">2</div><div class="text">loop {</div></div>
        <div class="line" data-index="2"><div class="num">3</div><div class="text">    led.set_high(); // Turn on the LED</div></div>
        <div class="line" data-index="3"><div class="num">4</div><div class="text">    Timer::after_millis(<span class="ms-val">5000</span>).await;</div></div>
        <div class="line" data-index="4"><div class="num">5</div><div class="text">    led.set_low(); // Turn off the LED</div></div>
        <div class="line" data-index="5"><div class="num">6</div><div class="text">    Timer::after_millis(<span class="ms-val">5000</span>).await;</div></div>
        <div class="line" data-index="6"><div class="num">7</div><div class="text">}</div></div>
      <div class="progress-wrap">
        <div class="progress" aria-hidden="true"><div id="bar" class="bar"></div></div>
        <div class="progress-info"><div id="progress-label">Idle</div><div id="ms-left">0 ms</div></div>
      </div>
      <div class="controls">
        <label>Interval (ms): <input id="interval" type="number" value="5000" min="50" step="50"></label>
        <button id="restart">Restart</button>
        <button id="pause">Pause</button>
        <button id="resume" style="display:none">Resume</button>
      </div>
    </div>
  </div>
</div>

<script>
document.addEventListener("DOMContentLoaded", () => {

  const lines = Array.from(document.querySelectorAll('.line'));
  const ledEl = document.getElementById('led');
  const ledState = document.getElementById('led-state');
  const bar = document.getElementById('bar');
  const progressLabel = document.getElementById('progress-label');
  const msLeft = document.getElementById('ms-left');
  const intervalInput = document.getElementById('interval');
  const restartBtn = document.getElementById('restart');
  const pauseBtn = document.getElementById('pause');
  const resumeBtn = document.getElementById('resume');
  const msValSpans = Array.from(document.querySelectorAll('.ms-val'));

  let intervalMs = Number(intervalInput.value) || 5000;
  let running = true;
  let paused = false;

  function setLed(on){
    if(on){
      ledEl.classList.add('on');
      ledState.textContent = 'HIGH';
    } else {
      ledEl.classList.remove('on');
      ledState.textContent = 'LOW';
    }
  }

  function highlight(index){
    lines.forEach(l => l.classList.toggle('current', Number(l.dataset.index) === index));
  }

  function sleep(ms){
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  // update the displayed ms inside code examples
  function updateCodeMs(v){
    msValSpans.forEach(el => el.textContent = v);
  }

  // animate a timer and update progress bar and labels
  function runTimer(ms, onUpdate){
    return new Promise(resolve => {
      let start = performance.now();
      let lastNow = start;
      function step(now){
        if(paused){
          // push the start forward so elapsed stops accumulating during pause
          start += (now - lastNow);
          lastNow = now;
          requestAnimationFrame(step);
          return;
        }
        const elapsed = now - start;
        const p = Math.min(1, elapsed / ms);
        onUpdate(p, Math.max(0, ms - Math.floor(elapsed)));
        if(p >= 1){
          resolve();
        } else {
          lastNow = now;
          requestAnimationFrame(step);
        }
      }
      requestAnimationFrame(step);
    });
  }

  async function mainLoop(){
    while(running){
      intervalMs = Number(intervalInput.value) || 5000;
      updateCodeMs(intervalMs);

      // line: led.set_high();
      highlight(2);
      await sleep(300);           // small pause so the highlight is visible
      setLed(true);
      progressLabel.textContent = 'Waiting after set_high()';

      // timer line
      highlight(3);
      bar.style.width = '0%';
      await runTimer(intervalMs, (p, left) => {
        bar.style.width = (p * 100) + '%';
        msLeft.textContent = left + ' ms';
      });

      // line: led.set_low();
      highlight(4);
      await sleep(300);           // small pause so the highlight is visible
      setLed(false);
      progressLabel.textContent = 'Waiting after set_low()';

      // timer line
      highlight(5);
      bar.style.width = '0%';
      await runTimer(intervalMs, (p, left) => {
        bar.style.width = (p * 100) + '%';
        msLeft.textContent = left + ' ms';
      });

      // end of loop - closing brace briefly
      highlight(6);
      progressLabel.textContent = 'Looping...';
      await sleep(120);

      bar.style.width = '0%';
      msLeft.textContent = '0 ms';
      progressLabel.textContent = 'Idle';

      // respect pause
      while(paused){
        await sleep(100);
      }
    }
  }

  // controls
  restartBtn.addEventListener('click', () => {
    paused = false;
    running = false;
    setTimeout(() => { running = true; mainLoop(); }, 50);
  });
  pauseBtn.addEventListener('click', () => {
    paused = true;
    pauseBtn.style.display = 'none';
    resumeBtn.style.display = 'inline-block';
    progressLabel.textContent = 'Paused';
  });
  resumeBtn.addEventListener('click', () => {
    paused = false;
    resumeBtn.style.display = 'none';
    pauseBtn.style.display = 'inline-block';
    progressLabel.textContent = 'Resuming...';
  });

  // update interval on input and reflect in code
  intervalInput.addEventListener('input', () => {
    intervalMs = Number(intervalInput.value) || 5000;
    updateCodeMs(intervalMs);
  });

  // initialize
  updateCodeMs(intervalMs);
  highlight(0);
  setLed(false);
  mainLoop();
});
</script>
