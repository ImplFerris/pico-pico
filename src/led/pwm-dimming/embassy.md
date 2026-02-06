# LED Dimming on Raspberry Pi Pico with Embassy

Let's create a dimming LED effect using PWM on the Raspberry Pi Pico with Embassy.

## Generate project using cargo-generate

By now you should be familiar with the steps. We use the cargo-generate command with our custom template, and when prompted, select Embassy as the HAL.

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

## Update Imports

Add the import below to bring the PWM types into scope:

```rust
use embassy_rp::pwm::{Pwm, SetDutyCycle};
```

## Initialize PWM

Let's set up the PWM for the LED. Use the first line for the onboard LED, or uncomment the second one if you want to use an external LED on GPIO 16.

```rust
// For Onboard LED
let mut pwm = Pwm::new_output_b(p.PWM_SLICE4, p.PIN_25, Default::default());

// For external LED connected on GPIO 16
// let mut pwm = Pwm::new_output_a(p.PWM_SLICE0, p.PIN_16, Default::default());
```

## Main logic

In the main loop, we create the fade effect by increasing the duty cycle from 0 to 100 percent and then bringing it back down. The small delay between each step makes the dimming smooth. You can adjust the delay and observe how the fade speed changes.

```rust
loop {
    for i in 0..=100 {
        Timer::after_millis(8).await;
        let _ = pwm.set_duty_cycle_percent(i);
    }

    for i in (0..=100).rev() {
        Timer::after_millis(8).await;
        let _ = pwm.set_duty_cycle_percent(i);
    }

    Timer::after_millis(500).await;
}
```

## The full code

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_rp as hal;
use embassy_rp::block::ImageDef;
use embassy_rp::pwm::{Pwm, SetDutyCycle};
use embassy_time::Timer;

//Panic Handler
use panic_probe as _;
// Defmt Logging
use defmt_rtt as _;

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());

    // For Onboard LED
    let mut pwm = Pwm::new_output_b(p.PWM_SLICE4, p.PIN_25, Default::default());

    // For external LED connected on GPIO 16
    // let mut pwm = Pwm::new_output_a(p.PWM_SLICE0, p.PIN_16, Default::default());

    loop {
        for i in 0..=100 {
            Timer::after_millis(8).await;
            let _ = pwm.set_duty_cycle_percent(i);
        }
        for i in (0..=100).rev() {
            Timer::after_millis(8).await;
            let _ = pwm.set_duty_cycle_percent(i);
        }
        Timer::after_millis(500).await;
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"led-dimming"),
    embassy_rp::binary_info::rp_program_description!(c"your program description"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```

## Clone the existing project

You can clone the project I created and navigate to the `external-led` folder:

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/led-dimming
```
