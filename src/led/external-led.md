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

