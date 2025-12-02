
# Blink an External LED on the Raspberry Pi Pico with Embedded Rust

Let's start by creating our project. We'll use cargo-generate and use the template we prepared for this book.

In your terminal, type:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.2.0
```

You will be asked a few questions:

1. For the project name, you can give anything. We will use external-led.

2. Next, it asks us to Select HAL. We should choose "Embassy".

3. Then, it will ask whether we want to enable defmt logging. This works only if we use a debug probe, so you can choose based on your setup.  Anyway we are not going to write any log in this exercise.

## Imports

Most of the required imports are already in the project template. For this exercise, we only need to add the `Output` struct and the `Level` enum from gpio:

```rust
use embassy_rp::gpio::{Level, Output};
```

While writing the main code, your editor will normally suggest missing imports. If something is not suggested or you see an error, check the full code section and add the missing imports from there.

## Main Logic

The code is almost the same as the quick start example. The only change is that we now use GPIO 13 instead of GPIO 25. GPIO 13 is where we connected the LED (through a resistor).

Let's add these code the main function :

```rust
let mut led = Output::new(p.PIN_13, Level::Low);

loop {
    led.set_high(); // Turn on the LED
    Timer::after_millis(500).await;

    led.set_low(); // Turn off the LED
    Timer::after_millis(500).await;
}
```

We are using the Output struct here because we want to send signals from the Pico to the LED. We set up GPIO 13 as an output pin and start it in the low (off) state.

> Note: If you want to read signals from a component (like a button or sensor), you'll need to configure the GPIO pin as Input instead.

Then we call set_high and set_low on the pin with a delay between them. This switches the pin between high and low, which turns the LED on and off.

## The Full code

Here is the complete code for reference:

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_rp as hal;
use embassy_rp::block::ImageDef;
use embassy_rp::gpio::{Level, Output};
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

    let mut led = Output::new(p.PIN_13, Level::Low);

    loop {
        led.set_high(); // Turn on the LED
        Timer::after_millis(500).await;

        led.set_low(); // Turn off the LED
        Timer::after_millis(500).await;
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"external-led"),
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
cd pico2-embassy-projects/external-led
```

## How to Run?

You refer the ["Running The Program"](../running.md) section
