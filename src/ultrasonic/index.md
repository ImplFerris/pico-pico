# Ultrasonic

In this section, we'll learn how to interface the HC-SR04 ultrasonic sensor with a Raspberry Pi Pico 2. Ultrasonic sensors measure distances by emitting ultrasonic sound waves and calculating the time taken for them to return after bouncing off an object.
 
 We will build a simple project that gradually increases the LED brightness using PWM, when the ultrasonic sensor detects an object distance of less than 30 cm.

<img style="display: block; margin: auto;width:500px" alt="pico2" src="../images/hc-sr04-ultrasonic.jpg"/>

## 🛠 Hardware Requirements
To complete this project, you will need:

- HC-SR04 Ultrasonic Sensor
- Breadboard
- Jumper wires
- External LED (You can also use the onboard LED, but you'll need to modify the code accordingly)

The HC-SR04 Sensor module has a transmitter and receiver. The module has Trigger and Echo pins which can be connected to the GPIO pins of a pico and other microcontrollers. When the receiver detects the returning sound wave, the Echo pin goes high for a duration equal to the time it takes for the wave to return to the sensor.

## Setup
- **VCC**: Connect to the 5V pin on the Pico to power the HC-SR04.
- **Trig**: Connect to GPIO 17 on the Pico to start the ultrasonic sound pulses.
- **Echo**: Connect to GPIO 16 on the Pico; this pin sends a pulse when it detects the reflected signal, and the pulse length shows how long the signal took to return.
- **GND**: Connect to the ground pin on the Pico.
- **LED**: Connect the anode (long leg) of the LED to GPIO 13, as in the [External LED setup](../blinky/external-led.md).
