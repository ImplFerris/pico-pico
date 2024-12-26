# PWM Peripheral in RP2350

The pico has a PWM peripheral with 12 PWM generators, known as slices. Each slice has two channels (A and B), providing a total of 24 PWM output channels available for use.

Refer the 1073th page of the [RP2350](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf) Datasheet for more information.

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

## Fading Effect

### Fading Up
The code below gradually increases the LED brightness by changing the duty cycle from 0 to 25,000, with a small delay between each step:
```rust
for i in LOW..=HIGH {
  delay.delay_us(8);
  let _ = channel.set_duty_cycle(i);
}
```
The delay ensures the LED brightens gradually.  We need the delay between them; otherwise, it would quickly jump from dim to bright. The delay allows for a smooth, noticeable "fading up" effect.

Dont' believe me! Adjust the delay to 0 and observe. You can increase the delay (eg: 25) and observe the fading effect.

Note: `set_duty_cycle` function under the hood writes the given value into CC register(Count compare value). 

### Fading Down
The following code reduce the LED brightness by decrementing the duty cycle from 25,000 to 0.
```rust
// Here rev is to reverse the iteration. so it goes from 25_000 to 0
for i in (LOW..=HIGH).rev() {  
    delay.delay_us(8);
    let _ = channel.set_duty_cycle(i);
}
```

### Pause
After fading up and down, the program pauses for 500 milliseconds before repeating the cycle, allowing the LED to rest briefly.

Play around by adjusting the `delay` and observe. You can even comment out one of the for loop and observe the effect. 

