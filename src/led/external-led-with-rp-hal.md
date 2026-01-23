# Blinky Example using rp-hal

In the previous section, we used Embassy. We keep the same circuit and wiring. For this example, we switch to rp-hal to show how both approaches look. You can choose Embassy if you want async support, or rp-hal if you prefer the blocking style. In this book, we will mainly use Embassy.

We will create a new project again with cargo-generate and the same template.

In your terminal, type:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

When it asks you to select HAL, choose "rp-hal" this time.

## Imports

The template already includes most imports. For this example, we need to add the OutputPin trait from embedded-hal:

```rust
// Embedded HAL trait for the Output Pin
use embedded_hal::digital::OutputPin;
```

This trait provides the set_high() and set_low() methods we'll use to control the LED.

## Main Logic

If you compare this with the Embassy version, there's not much difference in how the LED is toggled. The main difference is in how the delay works. Embassy uses async and await, which lets the program pause without blocking and allows other tasks to run in the background. rp-hal uses a blocking delay, which stops the program until the time has passed.

```rust
let mut led_pin = pins.gpio13.into_push_pull_output();

loop {
    led_pin.set_high().unwrap();
    timer.delay_ms(200);

    led_pin.set_low().unwrap();
    timer.delay_ms(200);
}
```

## Full code

```rust
#![no_std]
#![no_main]

use embedded_hal::delay::DelayNs;
use hal::block::ImageDef;
use rp235x_hal as hal;

//Panic Handler
use panic_probe as _;
// Defmt Logging
use defmt_rtt as _;

// Embedded HAL trait for the Output Pin
use embedded_hal::digital::OutputPin;

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();
/// External high-speed crystal on the Raspberry Pi Pico 2 board is 12 MHz.
/// Adjust if your board has a different frequency
const XTAL_FREQ_HZ: u32 = 12_000_000u32;

#[hal::entry]
fn main() -> ! {
    // Grab our singleton objects
    let mut pac = hal::pac::Peripherals::take().unwrap();

    // Set up the watchdog driver - needed by the clock setup code
    let mut watchdog = hal::Watchdog::new(pac.WATCHDOG);

    // Configure the clocks
    //
    // The default is to generate a 125 MHz system clock
    let clocks = hal::clocks::init_clocks_and_plls(
        XTAL_FREQ_HZ,
        pac.XOSC,
        pac.CLOCKS,
        pac.PLL_SYS,
        pac.PLL_USB,
        &mut pac.RESETS,
        &mut watchdog,
    )
    .ok()
    .unwrap();

    // The single-cycle I/O block controls our GPIO pins
    let sio = hal::Sio::new(pac.SIO);

    // Set the pins up according to their function on this particular board
    let pins = hal::gpio::Pins::new(
        pac.IO_BANK0,
        pac.PADS_BANK0,
        sio.gpio_bank0,
        &mut pac.RESETS,
    );

    let mut timer = hal::Timer::new_timer0(pac.TIMER0, &mut pac.RESETS, &clocks);

    let mut led_pin = pins.gpio13.into_push_pull_output();

    loop {
        led_pin.set_high().unwrap();
        timer.delay_ms(200);

        led_pin.set_low().unwrap();
        timer.delay_ms(200);
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [hal::binary_info::EntryAddr; 5] = [
    hal::binary_info::rp_cargo_bin_name!(),
    hal::binary_info::rp_cargo_version!(),
    hal::binary_info::rp_program_description!(c"your program description"),
    hal::binary_info::rp_cargo_homepage_url!(),
    hal::binary_info::rp_program_build_attribute!(),
];
```

## Clone the existing project

You can clone the project I created and navigate to the `external-led` folder:

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-rp-projects/external-led
```
