# Beep

In this exercise, we will generate a beeping sound with a 50% duty cycle, followed by a 0% duty cycle, creating a looping pattern of sound and silence. We will use the A4 note (440Hz frequency) for this. (If you're not familiar with the A4 note, please look up more information on musical notes.)

###  Get Top function
In the previous exercise (servo motor), we manually calculated and hardcoded the top value. In this exercise, we create a small function to calculate the top value based on the target frequency and div_int:
```rust
const fn get_top(freq: f64, div_int: u8) -> u16 {
    let result = 150_000_000. / (freq * div_int as f64);
    result as u16 - 1
}
```

### `div_int` value
We will be using 64 as `div_int`.
```rust
const PWM_DIV_INT: u8 = 64;
```

### Configure the GPIO 15 pin
Next, we need to configure the GPIO pin (GPIO 15) to output the PWM signal.
```rust
let pwm = &mut pwm_slices.pwm7;
pwm.enable();
pwm.set_div_int(PWM_DIV_INT);
pwm.channel_b.output_to(pins.gpio15);
```

### To Set a frequency 440Hz(A4 Note)
Now we calculate the top value required to generate the 440Hz frequency (A4 note) and set it for the PWM:
```rust
let top = get_top(440., PWM_DIV_INT);
pwm.set_top(top);
```

## Loop
Finally, we create a loop to alternate between a 50% duty cycle (beep) and a 0% duty cycle (silence). The loop repeats with a delay of 500 milliseconds between each change:
```rust
loop {
    pwm.channel_b.set_duty_cycle_percent(50).unwrap();
    timer.delay_ms(500);
    pwm.channel_b.set_duty_cycle(0).unwrap();
    timer.delay_ms(500);
}
```

## Clone the existing project
You can clone (or refer) project I created and navigate to the `beep` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/beep
```









