# PWM Peripheral in RP2350

The RP2350 has a PWM peripheral with 12 PWM generators called slices. Each slice contains two output channels (A and B), giving you a total of 24 PWM output channels. For detailed specifications, see page 1076 of the [RP2350 Datasheet](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf).


Let's have a quick look at some of the key concepts.

## PWM Generator (Slice)

A slice is the hardware block that generates PWM signals. Each of the 12 slices (PWM0–PWM11) is an independent timing unit with its own 16-bit counter, compare registers, control settings, and clock divider. This independence means you can configure each slice with different frequencies and resolutions.


## Channel

Each slice contains two output channels: **Channel A** and **Channel B**. Both channels share the same counter, so they run at the same frequency and are synchronized. However, each channel has its own compare register, allowing independent duty cycle control. This lets you generate two related but distinct PWM signals from a single slice.


## Mapping of PWM channels to GPIO Pins

Each GPIO pin connects to a specific slice and channel. You'll find the complete mapping table on page 1078 of the [RP2350 Datasheet](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf). For example, GP25 (the onboard LED pin) maps to PWM slice 4, channel B, labeled as **4B**.

<img style="display: block; margin: auto;" alt="pico2" src="../images/gpio-map-pwm-channels.png"/>

Initialize the PWM peripheral and get access to all slices:

```rust
let mut pwm_slices = hal::pwm::Slices::new(pac.PWM, &mut pac.RESETS);
```

Get a reference to PWM slice 4 for configuration:

```rust
let pwm = &mut pwm_slices.pwm4;
```

## Phase-Correct Mode

In standard PWM (fast PWM), the counter counts up from 0 to TOP, then immediately resets to 0. This creates asymmetric edges where the output changes at different points in the cycle.

Phase-correct PWM counts up to TOP, then counts back down to 0, creating a triangular waveform. The output switches symmetrically - once going up and once coming down. This produces centered pulses with edges that mirror each other, reducing electromagnetic interference and creating smoother transitions. The trade-off is that phase-correct mode runs at half the frequency of standard PWM for the same TOP value.

Configure PWM4 to operate in phase-correct mode for smoother output transitions.

```rust
pwm.set_ph_correct();
```

Get a mutable reference to channel B of PWM4 and direct its output to GPIO pin 25.
```rust
let channel = &mut pwm.channel_b;
channel.output_to(pins.gpio25);
```
