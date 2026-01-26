# Buzzer Beep Using PWM on Raspberry Pi Pico with Embedded Rust

In this exercise, we will generate a beep sound using a buzzer. The idea is similar to blinking an LED, but instead of toggling a GPIO HIGH and LOW, we control the PWM duty cycle.

We will repeatedly switch the PWM duty cycle between 50 percent and 0 percent, with a delay in between. When the duty cycle is 50 percent, the buzzer produces sound. When it is 0 percent, the sound stops. Repeating this creates a clear beep.

You can try this without changing the PWM frequency. In this example, we set the PWM frequency to 440.0 Hz, which corresponds to the `a4` musical note. You do not need to know anything about musical notes for this. The important point is that we generate a fixed-frequency tone and turn the sound on and off by changing the duty cycle.

## Create Project from template

We will start by creating a new project using the Embassy framework. As usual, generate the project from the template with `cargo-generate`:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

When prompted, give your project a name like "buzzer-beep" and choose "embassy" as the HAL. Enable `defmt` logging, if you have a debug probe so you can view logs also.

## Additional imports

As we have done before, we import the `SetDutyCycle` trait and the `Pwm` and `Config` types for PWM configuration.

```rust
use embassy_rp::pwm::{Config as PwmConfig, Pwm, SetDutyCycle};
```

## Calculate TOP

You can either calculate the TOP value manually or use the [calculator form](../core-concepts/pwm/frequency-to-pwm-top-divider.md) shown in the previous chapter. This time, we will take a different approach.

We will keep the divider fixed at 64 and use a `const fn` to calculate the TOP value. Since we are not using phase-correct mode or the fractional divider, the calculation is simple.

```rust
const fn get_top(freq: f64, div_int: u8) -> u16 {
    assert!(div_int != 0, "Divider must not be 0");

    let result = 150_000_000. / (freq * div_int as f64);

    assert!(result >= 1.0, "Frequency too high");
    assert!(
        result <= 65535.0,
        "Frequency too low: TOP exceeds 65534 max"
    );

    result as u16 - 1
}

const PWM_DIV_INT: u8 = 64;
const PWM_TOP: u16 = get_top(440., PWM_DIV_INT);
```

## Main logic

First, we configure the PWM with the calculated TOP value and the fixed divider. Then we create a PWM output for the buzzer. Inside the loop, we switch the duty cycle between 50 percent and 0 percent with a delay in between, which produces a repeating beep sound.

```rust
let mut pwm_config = PwmConfig::default();
pwm_config.top = PWM_TOP;
pwm_config.divider = PWM_DIV_INT.into();

let mut buzzer = Pwm::new_output_b(p.PWM_SLICE7, p.PIN_15, pwm_config);

loop {
    buzzer
        .set_duty_cycle_percent(50)
        .expect("50 is valid duty percentage");
    Timer::after_millis(1000).await;

    buzzer
        .set_duty_cycle_percent(0)
        .expect("0 is valid duty percentage");
    Timer::after_millis(1000).await;
}
```

If you want the beep to be shorter or faster, you can adjust the delay values.

## Clone the existing project

You can clone (or refer to) the project I created and navigate to the `buzzer-beep` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/buzzer-beep
```

## rp-hal version

If you want to see the same example implemented using rp-hal, you can find it here.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-rp-projects/buzzer-beep
```
