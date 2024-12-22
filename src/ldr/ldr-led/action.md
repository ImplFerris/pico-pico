# Action

We'll use the Embassy HAL for this exercise.

## Project from template

To set up the project, run:
```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git
```
When prompted, give your project a name, like "dracula-ldr" and select `embassy` as the HAL.

Then, navigate into the project folder:
```sh
cd PROJECT_NAME
# For example, if you named your project "dracula-ldr":
# cd dracula-ldr
```

## Interrupt Handler
Let's set up interrupt handling for the ADC.

```rust
use embassy_rp::adc::InterruptHandler;

bind_interrupts!(struct Irqs {
    ADC_IRQ_FIFO => InterruptHandler;
});
```
In simple terms, when the ADC completes a conversion and the result is ready, it triggers an interrupt. This tells the pico that the new data is available, so it can process the ADC value. The interrupt ensures that the pico doesn't need to constantly check the ADC, allowing it to respond only when new data is ready.

Read more about RP2350 interreupts in the [datasheet (82th page).](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf)
 

## Initialize the Embassy HAL
```rust
let p = embassy_rp::init(Default::default());
```

## Initialize the ADC
```rust
let mut adc = Adc::new(p.ADC, Irqs, Config::default());
```

## Configuring the ADC Pin and LED
We set up the ADC input pin (PIN_26) for reading an analog signal. 
Then we set up an output pin (PIN_15) to control an LED. The LED starts in the low state (Level::Low), meaning it will be off initially.

```rust
let mut p26 = Channel::new_pin(p.PIN_26, Pull::None);
let mut led = Output::new(p.PIN_15, Level::Low);
```

## Main loop
The logic is straightforward: read the ADC value, and if it's greater than 3800, turn on the LED; otherwise, turn it off.

```rust
loop {
    let level = adc.read(&mut p26).await.unwrap();
    if level > 3800 {
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
use embassy_rp::adc::{Adc, Channel, Config, InterruptHandler};
use embassy_rp::bind_interrupts;
use embassy_rp::block::ImageDef;
use embassy_rp::gpio::{Level, Output, Pull};
use embassy_time::Timer;
use {defmt_rtt as _, panic_probe as _};

#[link_section = ".start_block"]
#[used]
pub static IMAGE_DEF: ImageDef = ImageDef::secure_exe();

bind_interrupts!(struct Irqs {
    ADC_IRQ_FIFO => InterruptHandler;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());
    let mut adc = Adc::new(p.ADC, Irqs, Config::default());

    let mut p26 = Channel::new_pin(p.PIN_26, Pull::None);
    let mut led = Output::new(p.PIN_15, Level::Low);

    loop {
        let level = adc.read(&mut p26).await.unwrap();
        if level > 3800 {
            led.set_high();
        } else {
            led.set_low();
        }
        Timer::after_secs(1).await;
    }
}
```


## Clone the existing project
You can clone (or refer) project I created and navigate to the `dracula-ldr` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/dracula-ldr/
```
