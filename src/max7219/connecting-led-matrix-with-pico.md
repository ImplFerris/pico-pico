# Circuit

The MAX7219 display requires five connections to the Raspberry Pi Pico 2. This example uses SPI1 with GPIO 13, 14, and 15, but you can use any valid SPI pin set on your Pico.

<table>
  <thead>
    <tr>
      <th>Pico Pin</th>
      <th style="width: 250px; margin: 0 auto;">Wire</th>
      <th>MAX7219 Pin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 13 (SPI1 CS)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>CS / LOAD</td>
    </tr>
    <tr>
      <td>GPIO 14 (SPI1 SCK)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>CLK</td>
    </tr>
    <tr>
      <td>GPIO 15 (SPI1 MOSI)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>DIN</td>
    </tr>
    <tr>
      <td>VBUS (5 V)</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VCC</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GND</td>
    </tr>
  </tbody>
</table>
<br/>

<img style="display: block; margin: auto;" alt="Connecting Raspberry Pi Pico 2 with MAX7219 LED Matrix" src="./images/circuit-to-connect-max7219-led-matrix-with-raspberry-pi-pico.png"/>
