# Embedded Rust Code to Control an LED Based on Light Level

With the circuit assembled on your breadboard, let's write the code.

## Project from template

As usual, we are going to start by generating a new project from the template.

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```
When prompted, give your project a name, like "ldr-dracula" and select `embassy` as the HAL.

## Additional Imports

```rust
// For Interrupt Binding
use embassy_rp::adc::InterruptHandler;
use embassy_rp::bind_interrupts;

// For ADC
use embassy_rp::adc::{Adc, Channel, Config as AdcConfig};

// For LED
use embassy_rp::gpio::{Level, Output, Pull};
```

## Interrupt Handler for ADC

The RP2350's ADC has a FIFO (First-In-First-Out) buffer that stores conversion results (i.e, the measured voltage converted into a discrete ADC value). When this FIFO receives data, the hardware generates an interrupt signal called `ADC_IRQ_FIFO`.

In the Embassy library, the ADC interrupt handling logic is already implemented. We just need to bind this hardware interrupt so the ADC driver can use it.

Before we start reading values from the ADC, we need to set up this interrupt binding.

```rust
bind_interrupts!(struct Irqs {
    ADC_IRQ_FIFO => InterruptHandler;
});
```

## ADC Threshold

Before we write the main logic, we need to decide what counts as low light. We do this by defining a threshold value for the ADC reading.

```rust
const LDR_THRESHOLD: u16 = 200;
```

This threshold represents the light level at which the LED should turn on. When the ADC reading drops below this value, we treat the environment as dark. When the reading is above this value, we treat it as bright.

You may need to adjust this threshold depending on your room lighting, the LDR you are using, and the resistor values in the voltage divider.

## Creating the ADC Instance

Let's create the ADC instance.

```rust
let mut adc = Adc::new(p.ADC, Irqs, AdcConfig::default());
```

Here, we pass three things to the ADC constructor. We pass the ADC peripheral itself, the interrupt bindings we defined earlier, and a default configuration.

> [!Note]
> Interesting fact: the HAL does not actually do anything with `Irqs` at runtime when you pass it to the ADC constructor. It is only there at compile time to make sure you have declared the ADC interrupt binding. If you follow the new method, you will notice the parameter is named `_irq`, which makes it clear that it is not used.

## Configuring the ADC Pin

Next, we select which GPIO pin will be used as the ADC input.

```rust
let mut adc_pin = Channel::new_pin(p.PIN_28, Pull::None);
```

## Configuring the LED Output

Now we configure the GPIO pin that drives the LED.

```rust
let mut led = Output::new(p.PIN_15, Level::Low);
```

## Main loop

The logic is straightforward: read the ADC value, and if it's lesser than threshold we defined earlier, we turn on the LED; otherwise, turn it off.

```rust
loop {
    let adc_reading = adc
        .read(&mut adc_pin)
        .await
        .expect("Unable to read the adc value");
    defmt::info!("ADC value: {}", adc_reading);

    if adc_reading < LDR_THRESHOLD {
        led.set_high();
    } else {
        led.set_low();
    }

    Timer::after_secs(1).await;
}
```

## The full code

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_rp as hal;
use embassy_rp::block::ImageDef;
use embassy_time::Timer;

// Interrupt Binding
use embassy_rp::adc::InterruptHandler;
use embassy_rp::bind_interrupts;

// ADC
use embassy_rp::adc::{Adc, Channel, Config as AdcConfig};

// For LED
use embassy_rp::gpio::{Level, Output, Pull};

//Panic Handler
use panic_probe as _;
// Defmt Logging
use defmt_rtt as _;

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

bind_interrupts!(struct Irqs {
    ADC_IRQ_FIFO => InterruptHandler;
});

const LDR_THRESHOLD: u16 = 200;

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());

    let mut adc = Adc::new(p.ADC, Irqs, AdcConfig::default());

    let mut adc_pin = Channel::new_pin(p.PIN_28, Pull::None);
    let mut led = Output::new(p.PIN_15, Level::Low);

    loop {
        let adc_reading = adc
            .read(&mut adc_pin)
            .await
            .expect("Unable to read the adc value");
        defmt::info!("ADC value: {}", adc_reading);
        if adc_reading < LDR_THRESHOLD {
            led.set_high();
        } else {
            led.set_low();
        }
        Timer::after_secs(1).await;
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"ldr-dracula"),
    embassy_rp::binary_info::rp_program_description!(c"your program description"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```

## Clone the existing project
You can clone (or refer) project I created and navigate to the `ldr-dracula` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/ldr-dracula/
```
