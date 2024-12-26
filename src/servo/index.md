# Servo Motor and PWM

In this section, we'll connect an SG90 Micro Servo Motor to the Pico 2 and control its rotation using PWM. The servo will move in a loop, rotating from 0 degrees to 90 degrees, and then to 180 degrees.

Before moving forward, make sure you've read the [PWM introduction](../core-concepts/pwm/index.md) in the Blink LED section.

## Hardware Requirements
- **SG90 Micro Servo Motor**
- **Jumper Wires**:
  - **Female-to-Male**(or Male to Male depending on how you are connecting) jumper wires for connecting the Pico 2 to the servo motor pins (Ground, Power, and Signal).

## Connection Overview
1. **Ground (GND)**: Connect the servo's GND pin (typically the **brown** wire, though it may vary) to any ground pin on the Pico 2.
2. **Power (VCC)**: Connect the servo's VCC pin (usually the **red** wire) to the Pico 2's 5V (or 3.3V if required by your setup) power pin.
3. **Signal (PWM)**: Connect the servo's control (signal) pin to **GPIO9** on the Pico 2, configured for PWM. This is commonly the **orange** wire (may vary).

<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>Pico Pin</th>
      <th style="width: 250px; margin: 0 auto;">Wire</th>
      <th>Servo Motor</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>VBUS</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Power (Red Wire)</td>
      <td>Supplies 5V power to the servo.</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire brown" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Ground (Brown Wire)</td>
      <td>Connects to ground.</td>
    </tr>
    <tr>
      <td>GPIO 16</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Signal (Orange Wire)</td>
      <td>Receives PWM signal to control the servo's position.</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;" alt="pico2" src="./images/pico-servo-circuit.png"/>

