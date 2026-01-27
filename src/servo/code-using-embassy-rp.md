# Servo Motor Control on Raspberry Pi Pico Using Embassy and Rust

In this section, we will create a simple program that moves the servo horn from 0 to 90 to 180 and then back to 0. This basic movement is enough to understand how PWM controls a servo. Once you are comfortable with the idea, you can experiment further and build more interesting applications.

We will start by creating a new project using the Embassy framework. After that, we wll build the same project again using `rp-hal`. As usual, generate the project from the template with `cargo-generate`:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

When prompted, give your project a name like "servo-motor" and choose "embassy" as the HAL. Enable `defmt` logging, if you have a debug probe so you can view logs also.

## Additional Imports

In addition to the usual boilerplate imports, you'll need to add these specific imports to your project.

```rust
// For PWM
use embassy_rp::pwm::{Config as PwmConfig, Pwm, SetDutyCycle};
```

## PWM Config

In the LED dimming chapter, we left the PWM configuration at its default values. That was sufficient there, because only the duty cycle mattered.

This time, we cannot do that. For servo control, we have to configure the TOP value and the divider ourselves so that the PWM frequency comes out to 50 Hz, based on the values we calculated earlier.

Here, I am using the manually calculated TOP and divider values directly in the code instead of using the calculator form. The divider I am using is a whole number, so I can simply convert it using the `into()` method. If the divider had a fractional part, I would need to use the `fixed` crate, which we already looked at earlier. To keep things simple, I am sticking to the integer version for now.

```rust
const PWM_DIV_INT: u8 = 64;
const PWM_TOP: u16 = 46_874;
```

> [!Note]
> You can also try this with a fractional divider. We already looked at the code snippet for that earlier, so you can reuse it and experiment with fractional values if you want.

Once we have those values, we just apply them to the PWM configuration like this.

```rust
let mut servo_config: PwmConfig = Default::default();
servo_config.top = PWM_TOP;
servo_config.divider = PWM_DIV_INT.into();
```

## Initialize PWM

Once the PWM configuration is ready, the next step is to create a PWM output and bind it to the GPIO pin connected to the servo signal wire.

In our case, we are using the GPIO 15. Feel free to change these if your wiring is different.

```rust
let mut servo = Pwm::new_output_b(p.PWM_SLICE7, p.PIN_15, servo_config);
```

## Main loop

Now we move on to the main loop. Here, we simply change the duty cycle value, wait for a short delay, and then move to the next position.

```rust
loop {
    // Move servo to 0° position (2.5 % duty cycle = 25/1000)
    servo
        .set_duty_cycle_fraction(25, 1000)
        .expect("invalid min duty cycle");

    Timer::after_millis(1000).await;

    // 90° position (7.5 % duty cycle)
    servo
        .set_duty_cycle_fraction(75, 1000)
        .expect("invalid half duty cycle");

    Timer::after_millis(1000).await;

    // 180° position (12 % duty cycle)
    servo
        .set_duty_cycle_fraction(120, 1000)
        .expect("invalid max duty cycle");

    Timer::after_millis(1000).await;
}
```

If everything works, you should see the servo horn move to the first position, pause briefly, move to the next position, and then move to the final position before returning back again.

## Clone the existing project

You can clone (or refer) project I created and navigate to the `servo-motor` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/servo-motor
```

## Debugging

If your servo is not moving, start by checking the wiring. Make sure the signal wire is connected to the correct GPIO pin, the servo has a proper power source, and the ground is shared with the Pico.

Next, double check that the code was flashed correctly and that the program is actually running on the board. If you are using a debug probe with `defmt` enabled, the log output can help confirm this.

If everything looks correct and the servo still does not move as expected, the most likely reason is that your servo uses slightly different pulse widths for each position. In that case, refer to the datasheet for your specific servo model, or check the manufacturer or vendor website if they provide timing information. You may need to adjust the duty cycle values to match your servo.

Do not worry if this does not work perfectly the first time. This is one of the things I struggled with when I started as well. I have tried my best to explain the calculations and the reasoning behind them clearly. I hope this helps.

## The Full Code

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_rp as hal;
use embassy_rp::block::ImageDef;
use embassy_time::Timer;

//Panic Handler
use panic_probe as _;
// Defmt Logging
use defmt_rtt as _;

// PWM
use embassy_rp::pwm::{Config as PwmConfig, Pwm, SetDutyCycle};

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

const PWM_DIV_INT: u8 = 64;
const PWM_TOP: u16 = 46_874;

// Alternative method:
// const TOP: u16 = PWM_TOP + 1;
// const MIN_DUTY: u16 = (TOP as f64 * (2.5 / 100.)) as u16;
// const HALF_DUTY: u16 = (TOP as f64 * (7.5 / 100.)) as u16;
// const MAX_DUTY: u16 = (TOP as f64 * (12. / 100.)) as u16;

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());

    let mut servo_config: PwmConfig = Default::default();
    servo_config.top = PWM_TOP;
    servo_config.divider = PWM_DIV_INT.into();

    let mut servo = Pwm::new_output_b(p.PWM_SLICE7, p.PIN_15, servo_config);

    loop {
        // Move servo to 0° position (2.5 % duty cycle = 25/1000)
        servo
            .set_duty_cycle_fraction(25, 1000)
            .expect("invalid min duty cycle");
        Timer::after_millis(1000).await;

        // 90° position (7.5 % duty cycle)
        servo
            .set_duty_cycle_fraction(75, 1000)
            .expect("invalid half duty cycle");
        Timer::after_millis(1000).await;

        // 180° position (12 % duty cycle)
        servo
            .set_duty_cycle_fraction(120, 1000)
            .expect("invalid max duty cycle");
        Timer::after_millis(1000).await;
    }

    // Alternative method
    // loop {
    //     servo
    //         .set_duty_cycle(MIN_DUTY)
    //         .expect("invalid min duty cycle");
    //     Timer::after_millis(1000).await;

    //     servo
    //         .set_duty_cycle(HALF_DUTY)
    //         .expect("invalid half duty cycle");
    //     Timer::after_millis(1000).await;

    //     servo
    //         .set_duty_cycle(MAX_DUTY)
    //         .expect("invalid max duty cycle");
    //     Timer::after_millis(1000).await;
    // }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"servo-motor"),
    embassy_rp::binary_info::rp_program_description!(c"your program description"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```
