
# Connecting Buzzer with Raspberry Pi Pico

We will connect GPIO 15 to the buzzer's positive (signal) pin and the Pico's GND to the buzzer's ground pin. You are free to use a different GPIO pin if needed.

<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>Pico Pin</th>
      <th style="width: 250px; margin: 0 auto;">Wire</th>
      <th>Buzzer Pin</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 15</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Positive Pin</td>
      <td>Receives PWM signals to produce sound.</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Ground Pin</td>
      <td>Connects to ground.</td>
    </tr>
  </tbody>
</table>

<img style="display: block; margin: auto;" alt="pico2" src="./images/pico-buzzer-circuit.png"/>
