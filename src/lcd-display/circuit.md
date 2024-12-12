# Circuit

## Connecting LCD Display (LCD1602) to the Raspberry Pi Pico

We will be using parallel interface in 4bit mode. Remaining Pins like D0 to D3 won't be connected.

<table>
  <thead>
    <tr>
      <th style="width: 250px;">LCD Pin</th>
      <th style="width: 250px; text-align: center;">Wire</th>
      <th>Pico Pin</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>VSS</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GND</td>
      <td>Ground</td>
    </tr>
    <tr>
      <td>VDD</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>VBUS (5V)</td>
      <td>Power Supply</td>
    </tr>
    <tr>
      <td>V<sub>O</sub></td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire brown" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GND or Potentiometer</td>
      <td>You can use Potentiometer or resistors to adjust the contrast. I placed two 1K resistors in between Ground and V<sub>O</sub></td>
    </tr>
    <tr>
      <td>RS</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 16</td>
      <td>Register Select (0 = command, 1 = data)</td>
    </tr>
    <tr>
      <td>RW</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GND</td>
      <td>Read/Write. Set '0' to write to display. If you want to read from display, set '1'</td>
    </tr>
    <tr>
      <td>EN</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 17</td>
      <td>Enable</td>
    </tr>
    <tr>
      <td>D4</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 18</td>
      <td>Data Bit 4</td>
    </tr>
    <tr>
      <td>D5</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 19</td>
      <td>Data Bit 5</td>
    </tr>
    <tr>
      <td>D6</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire brown" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 20</td>
      <td>Data Bit 6</td>
    </tr>
    <tr>
      <td>D7</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire purple" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GPIO 21</td>
      <td>Data Bit 7</td>
    </tr>
    <tr>
      <td>A</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>3V3(OUT)</td>
      <td>LED Backlight +</td>
    </tr>
    <tr>
      <td>K</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>GND</td>
      <td>LED Backlight -</td>
    </tr>
  </tbody>
</table>
<br/>
<img style="display: block; margin: auto;" alt="lcd1602" src="./images/connecting-pico-with-lcd-display-lcd1602.jpg"/>
