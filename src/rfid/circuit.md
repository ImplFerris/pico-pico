## Circuit
 
The introduction has become quite lengthy, so we will move the circuit diagram for connecting the Pico to the RFID reader to a separate page. Additionally, there are more pins that involved in this than any of the previous components we've used so far.

## Pinout diagram of RC522
There are 8 pins in the RC522 RFID module.
<a href="./images/rc522-pinout.jpg"><img style="display: block; margin: auto;" alt="pinout diagram of RC522" src="./images/rc522-pinout.jpg"/></a>

<table style="border-collapse: collapse; width: 100%; border: 1px solid black;">
  <tr style="border: 1px solid black;">
    <th style="background-color: #009B77; border: 1px solid black;">Pin</th>
    <th style="border: 1px solid black;">SPI Function</th>
    <th style="border: 1px solid black;">I²C Function</th>
    <th style="border: 1px solid black;">UART Function</th>
    <th style="border: 1px solid black;">Description</th>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #ff0000; color: white; border: 1px solid black;">3.3V</td>
    <td style="border: 1px solid black;">Power</td>
    <td style="border: 1px solid black;">Power</td>
    <td style="border: 1px solid black;">Power</td>
    <td style="border: 1px solid black;">Power supply (3.3V).</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #000; color: white; border: 1px solid black;">GND</td>
    <td style="border: 1px solid black;">Ground</td>
    <td style="border: 1px solid black;">Ground</td>
    <td style="border: 1px solid black;">Ground</td>
    <td style="border: 1px solid black;">Ground connection.</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #7B3F00; color: white; border: 1px solid black;">RST</td>
    <td style="border: 1px solid black;">Reset</td>
    <td style="border: 1px solid black;">Reset</td>
    <td style="border: 1px solid black;">Reset</td>
    <td style="border: 1px solid black;">Reset the module.</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #EFEFEF; border: 1px solid black;color:black;">IRQ</td>
    <td style="border: 1px solid black;">Interrupt (optional)</td>
    <td style="border: 1px solid black;">Interrupt (optional)</td>
    <td style="border: 1px solid black;">Interrupt (optional)</td>
    <td style="border: 1px solid black;">Interrupt Request (IRQ) informs the microcontroller when an RFID tag is detected. Without using IRQ, the microcontroller would need to constantly poll the module.</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #FFC000; color: black; border: 1px solid black;">MISO</td>
    <td style="border: 1px solid black;">Master-In-Slave-Out</td>
    <td style="border: 1px solid black;">SCL</td>
    <td style="border: 1px solid black;">TX</td>
    <td style="border: 1px solid black;">In SPI mode, it acts as Master-In-Slave-Out (MISO). In I²C mode, it functions as the clock line (SCL). In UART mode, it acts as the transmit pin (TX).</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #008000; color: white; border: 1px solid black;">MOSI</td>
    <td style="border: 1px solid black;">Master-Out-Slave-In</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">In SPI mode, it acts as Master-Out-Slave-In (MOSI).</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #0F52BA; color: white; border: 1px solid black;">SCK</td>
    <td style="border: 1px solid black;">Serial Clock</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">-</td>
    <td style="border: 1px solid black;">In SPI mode, it acts as the clock line that synchronizes data transfer.</td>
  </tr>
  <tr style="border: 1px solid black;">
    <td style="background-color: #FF5F1F; color: white; border: 1px solid black;">SDA</td>
    <td style="border: 1px solid black;">Slave Select (SS)</td>
    <td style="border: 1px solid black;">SDA</td>
    <td style="border: 1px solid black;">RX</td>
    <td style="border: 1px solid black;">In SPI mode, it acts as the Slave select (SS, also referred as Chip Select). In I²C mode, it serves as the data line (SDA). In UART mode, it acts as the receive pin (RX).</td>
  </tr>
</table>

## Connecting the RFID Reader to the Raspberry Pi Pico
To establish communication between the Raspberry Pi Pico and the RFID Reader, we will use the SPI (Serial Peripheral Interface) protocol.  The SPI interface can handle data speed up to 10 Mbit/s. We wont be utilizing the following Pins: RST, IRQ at the moment.

<table>
  <thead>
    <tr>
      <th>Pico Pin</th>
      <th style="width: 250px; margin: 0 auto;">Wire</th>
      <th>RFID Reader Pin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>3.3V</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>3.3V</td>
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
    <!-- <tr>
      <td>GPIO 22</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire brown" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>RST</td>
    </tr> -->
    <tr>
      <td>GPIO 4</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire yellow" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>MISO</td>
    </tr>
    <tr>
      <td>GPIO 5</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire orange" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>SDA</td>
    </tr>
    <tr>
      <td>GPIO 6</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire blue" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>SCK</td>
    </tr>
    <tr>
      <td>GPIO 7</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire green" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>MOSI</td>
    </tr>
  </tbody>
</table>

<a href="./images/connecting-pico-with-mfrc522.jpg"><img style="display: block; margin: auto;" alt="pinout diagram of RC522" src="./images/connecting-pico-with-mfrc522.jpg"/></a>
