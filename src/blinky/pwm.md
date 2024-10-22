
## What is PWM?
PWM stands for **Pulse Width Modulation**. It is a technique used to control the amount of power delivered to a device by adjusting the width of the pulses in a signal.

In PWM, a digital signal switches between **on** and **off** states at a high frequency. The **duty cycle** of the signal determines how long it stays on compared to how long it stays off. 

- **Duty Cycle**: The percentage of time the signal is on during one cycle. 
  - For example:
    - 100% duty cycle means the signal is always on.
    - 50% duty cycle means the signal is on half the time and off half the time.
    - 0% duty cycle means the signal is always off.

In the previous example code, we use PWM to fade an LED.

### Fading Up
The first loop gradually increases the LED brightness from `LOW` (0) to `HIGH` (25,000). The `delay.delay_us(8)` creates a short pause between each increase, allowing the LED to visibly brighten.

### Fading Down
The second loop decreases the LED brightness back down to `LOW` (0), again using the same delay to make the transition smooth.

### Pause
After fading up and down, the program pauses for 500 milliseconds before repeating the cycle, allowing the LED to rest briefly.

Play around by adjusting the `delay` and observe. You can even comment out one of the for loop and observe the effect. 


## PWM Peripheral in RP2350
The PWM peripheral is responsible for generating PWM signals. The Pico 2 features 12 PWM generators, known as slices, with each slice having two channels(A/B). This configuration results in a total of 24 PWM output channels available for use.

**Mapping of PWM channels to GPIO Pins**: 

You can find the table on page 1073 of the [RP2350](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf) Datasheet for more information. You have to refer this table when using a specific GPIO, as each GPIO corresponds to a particular slice. For instance, using GP25 (for the LED) means you are working with output 4B, which is the B output of the fourth slice.

<img style="display: block; margin: auto;" alt="pico2" src="../images/gpio-map-pwm-channels.png"/>

Initialize the PWM slices by creating an instance using the PAC's PWM peripheral and reset control.
```rust
let mut pwm_slices = hal::pwm::Slices::new(pac.PWM, &mut pac.RESETS);
```

Retrieve a mutable reference to PWM4 from the initialized PWM slices for further configuration.
```rust
let pwm = &mut pwm_slices.pwm4;
```

Configure PWM4 to operate in phase-correct mode for smoother output transitions.  (You can refer the [secrets arudion PWM](https://docs.arduino.cc/tutorials/generic/secrets-of-arduino-pwm/) if you want to know what is phase-correct)
```rust
pwm.set_ph_correct();
```

Get a mutable reference to channel B of PWM4 and direct its output to GPIO pin 25.
```rust
let channel = &mut pwm.channel_b;
channel.output_to(pins.gpio25);
```
