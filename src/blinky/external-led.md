# Blinking an External LED

In this section, we’ll use an external LED to blink.

You'll need some basic electronic components

**Hardware Requirements**
- LED
- Resistor
- Jumper wires

Refer the [Raspberry pi guide](https://projects.raspberrypi.org/en/projects/introduction-to-the-pico/7) for the hardware setup. I actually used it with breadboard instead.

## Components Overview

1. LED
An LED (Light Emitting Diode) emits light when an electric current flows through it. It only lights up if connected correctly. Anode (long leg) of the LED should be connected to positive; We will connect it to the GP13.  This Cathode (short leg) should be connected to to ground(GND).

Note: Always use a resistor with LEDs. Without it, excessive current could damage the LED or even cause it to explode.

2. Resistors
A resistor regulates the flow of current in a circuit. Its value is measured in Ohms (Ω). We use resistors to limit current and protect components, such as LEDs. Resistors are labeled with color bands that represent their resistance value.

<img style="display: block; margin: auto;" alt="pico2" src="../images/pico-external-led.png"/>

## Code
There isn't much change in the code. We showcase how to blink an external LED connected to GPIO 13.  The code uses a push-pull output configuration to control the LED state. By setting the pin high, the LED is turned on, and by setting it low, the LED is turned off. This process is repeated in an infinite loop with a delay of 200 milliseconds between each state change, allowing the LED to blink at a consistent rate. 

```rust
let mut led_pin = pins.gpio13.into_push_pull_output();

loop {
    led_pin.set_high().unwrap();
    delay.delay_ms(200);
    led_pin.set_low().unwrap();
    delay.delay_ms(200);
}
```

## Clone the existing project
You can clone the blinky project I created and navigate to the `external-led` folder to run this version of the blink program:

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/external-led
```

## How to Run?

You refer the ["Running The Program"](../running.md) section
