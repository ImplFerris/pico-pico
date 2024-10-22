# Blink LED

In this section, we'll learn how to blink an LED using the Raspberry Pi Pico 2. In embedded system programming, blinking an LED is the equivalent of "Hello, World!"

## How to make the LED blink?

The onboard LED of the Pico is connected to GPIO pin 25 (based on the datasheet). To make it blink, we toggle the pin between high and low states at regular intervals. This turns the LED on and off, producing a blinking effect.

<img style="display: block; margin: auto;" alt="pico2" src="../images/pico2-board.png"/>
