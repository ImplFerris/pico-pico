# Understanding Joystick Positions Using ADC Values on Raspberry Pi Pico

In this chapter, we are going to print the ADC values from the joystick whenever they change. We will read two analog signals, vrx and vry, which are connected to the joystick axes, and we will also print the values when the joystick button is pressed.

The goal here is simple. We want to see how the physical movement of the joystick translates into numeric ADC values. Once you see how the values change as you move it around, you can extend the idea and use it in projects such as controlling robots, RC vehicles, drones, pan-tilt camera mounts and more.

## Project from template

We will start by generating a new project using the template.

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

When prompted, give your project a name, for example joystick-adc, and select embassy as the HAL.

If you have a debug probe, you can enable defmt and logging will be straightforward. If you do not have a debug probe, disable defmt and use USB serial output, just like we did in the previous chapter. We will not repeat those steps here so that we can stay focused on the joystick and ADC setup.

## Additional imports

Add the following imports to your main.rs file. These are needed for ADC access, GPIO input, and interrupt handling.

```rust
// For ADC
use embassy_rp::adc::{Adc, Channel, Config as AdcConfig};
use embassy_rp::gpio::{Input, Pull};

// Interrupt Binding
use embassy_rp::adc;
use embassy_rp::bind_interrupts;
```

## ADC Interrupt Binding

Next, we bind the ADC FIFO interrupt.

```rust
bind_interrupts!(struct Irqs {
    ADC_IRQ_FIFO => adc::InterruptHandler;
});
```

## ADC and Joystick Setup

Now we initialize the ADC peripheral and set up the joystick pins. The vrx and vry signals are connected to GPIO pins that support ADC. And we also set up the joystick button as a regular input.

```rust
// ADC Setup
let mut adc = Adc::new(p.ADC, Irqs, AdcConfig::default());

let mut vrx_pin = Channel::new_pin(p.PIN_27, Pull::None);
let mut vry_pin = Channel::new_pin(p.PIN_26, Pull::None);
let button = Input::new(p.PIN_15, Pull::Up);
```

## Tracking Previous Values

Before we enter the main loop, we keep a few variables to track previous readings. This lets us detect when something actually changes instead of printing values continuously.

```rust
let mut prev_vrx: u16 = 0;
let mut prev_vry: u16 = 0;
let mut print_vals = true;
let mut prev_btn_state = false;
```

We store the previous X and Y readings, keep a small flag to control when values should be printed, and also remember the previous button state so we can detect a button press event.

## Main loop

Inside the main loop, we continuously read the ADC values from the two joystick pins. If an ADC read fails, we skip that iteration and try again.

```rust
loop {
    let Ok(vry) = adc.read(&mut vry_pin).await else {
        continue;
    };
    let Ok(vrx) = adc.read(&mut vrx_pin).await else {
        continue;
    };
```

### Detecting Meaningful Changes

Analog joysticks are not perfectly stable. Even when untouched, the readings can fluctuate slightly. To keep the output readable, we only react when the change is large enough to matter.


```rust
    if vrx.abs_diff(prev_vrx) > 100 {
        prev_vrx = vrx;
        print_vals = true;
    }

    if vry.abs_diff(prev_vry) > 100 {
        prev_vry = vry;
        print_vals = true;
    }
```

When either axis changes beyond the threshold, we update the stored value and mark that the readings should be printed.

### Printing ADC Values

If the print_vals flag is set, we print the current joystick position and then clear the flag.

```rust
    if print_vals {
        print_vals = false;

        info!("X: {} Y: {}", vrx, vry);
    }
```

### Handling the Button

Next, we read the joystick button. Because it uses a pull-up resistor, a pressed button reads as low.

```rust
    let btn_state = button.is_low();
    if btn_state && !prev_btn_state {
        info!("Button Pressed");

        print_vals = true;
    }
    prev_btn_state = btn_state;
```

When the button is pressed, we mark the ADC values to be printed. This lets us see the exact joystick position at the moment of the press.


Finally, we add a short delay at the end of the loop.
```rust
    Timer::after_millis(100).await;
}
```

## The full code

```rust
#![no_std]
#![no_main]

use defmt::info;
use embassy_executor::Spawner;
use embassy_rp as hal;
use embassy_rp::block::ImageDef;
use embassy_time::Timer;

// For ADC
use embassy_rp::adc::{Adc, Channel, Config as AdcConfig};
use embassy_rp::gpio::{Input, Pull};

// Interrupt Binding
use embassy_rp::adc;
use embassy_rp::bind_interrupts;

//Panic Handler
use panic_probe as _;
// Defmt Logging
use defmt_rtt as _;

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

bind_interrupts!(struct Irqs {
    ADC_IRQ_FIFO => adc::InterruptHandler;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());

    // ADC Setup
    let mut adc = Adc::new(p.ADC, Irqs, AdcConfig::default());

    let mut vrx_pin = Channel::new_pin(p.PIN_27, Pull::None);
    let mut vry_pin = Channel::new_pin(p.PIN_26, Pull::None);
    let button = Input::new(p.PIN_15, Pull::Up);

    let mut prev_vrx: u16 = 0;
    let mut prev_vry: u16 = 0;
    let mut print_vals = true;
    let mut prev_btn_state = false;

    loop {
        let Ok(vry) = adc.read(&mut vry_pin).await else {
            continue;
        };
        let Ok(vrx) = adc.read(&mut vrx_pin).await else {
            continue;
        };

        if vrx.abs_diff(prev_vrx) > 100 {
            prev_vrx = vrx;
            print_vals = true;
        }

        if vry.abs_diff(prev_vry) > 100 {
            prev_vry = vry;
            print_vals = true;
        }

        if print_vals {
            print_vals = false;

            info!("X: {} Y: {}", vrx, vry);
        }

        let btn_state = button.is_low();
        if btn_state && !prev_btn_state {
            info!("Button Pressed");

            print_vals = true;
        }
        prev_btn_state = btn_state;

        Timer::after_millis(100).await;
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recommended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"joystick-adc"),
    embassy_rp::binary_info::rp_program_description!(c"your program description"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```


## Clone the existing project

You can clone (or refer) project I created and navigate to the `joystick-adc` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/joystick/joystick-adc
```
