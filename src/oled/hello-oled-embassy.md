# Hello OLED

We are going to keep things simple. We will just display "Hello, Rust!" on the OLED display. We will first use Embassy, then we will do the same using rp-hal.

## Create Project

As usual, generate the project from the template with cargo-generate:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

When prompted, give your project a name like "hello-oled" and choose "embassy" as the HAL. Enable defmt logging, if you have a debug probe so you can view logs also.

## Update Dependencies

Add the following lines to your Cargo.toml under dependencies:

```toml
embedded-graphics = "0.8.1"
ssd1306 = { version = "0.10.0", features = ["async"] }
```

We will enable the `async` feature so the ssd1306 driver can be used with Embassy async I2C. You can also use it without this feature and use Embassy I2C in blocking mode.

## Additional imports

Add these imports at the top of your main.rs:

```rust
// Interrupt Binding
use embassy_rp::peripherals::I2C0;
use embassy_rp::{bind_interrupts, i2c};

// I2C
use embassy_rp::i2c::{Config as I2cConfig, I2c};

// OLED
use ssd1306::{I2CDisplayInterface, Ssd1306Async, prelude::*};

// Embedded Graphics
use embedded_graphics::{
    mono_font::{MonoTextStyleBuilder, ascii::FONT_6X10},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
    text::{Baseline, Text},
};
```

## Bind I2C Interrupt

We discussed this in detail in the interrupts section, so you should already be familiar with what it does. This binds the `I2C0_IRQ` interrupt to the Embassy I2C interrupt handler for `I2C0`.

```rust
bind_interrupts!(struct Irqs {
    I2C0_IRQ => i2c::InterruptHandler<I2C0>;
});
```

## Initialize I2C

First, we need to set up the I2C bus to communicate with the display.

```rust
let sda = p.PIN_16;
let scl = p.PIN_17;

let mut i2c_config = I2cConfig::default();
i2c_config.frequency = 400_000; //400kHz

let i2c_bus = I2c::new_async(p.I2C0, scl, sda, Irqs, i2c_config);
```

We have connected the OLED's SDA line to Pin 16 and the SCL line to Pin 17. Throughout this chapter we will keep using these same pins. If you have connected your display to a different valid I2C pair, adjust the code to match your wiring.

We are using the new_async method to create an I2C instance in async mode. This allows I2C transfers to await instead of blocking the CPU. We use a 400 kHz bus speed, which is commonly supported by SSD1306 displays.

## Initialize Display

Now we create the display interface and initialize it:

```rust
let i2c_interface = I2CDisplayInterface::new(i2c_bus);

let mut display = Ssd1306Async::new(i2c_interface, DisplaySize128x64, DisplayRotation::Rotate0)
    .into_buffered_graphics_mode();
```

I2CDisplayInterface::new(i2c_bus) wraps the async I2C bus so it can be used by the SSD1306 driver. It uses the default I2C address 0x3C, which is standard for most SSD1306 modules.

We create the display instance by specifying a 128x64 display and the default orientation. We also enable buffered graphics mode so we can draw into a RAM buffer using embedded-graphics.

```rust
display
    .init()
    .await
    .expect("failed to initialize the display");
```
Finally, display.init() sends initialization commands to the display hardware. This wakes up the display and configures it properly.

## Writing Text

Before we can draw text, we need to define how the text should look:

```rust
 let text_style = MonoTextStyleBuilder::new()
        .font(&FONT_6X10)
        .text_color(BinaryColor::On)
        .build();
```

This creates a text style using FONT_6X10, a built-in monospaced font that's 6 pixels wide and 10 pixels tall. We set BinaryColor::On to display white pixels on our black background since the OLED is monochrome.

Now let's draw the text to the display's buffer:

```rust
defmt::info!("sending text to display");
Text::with_baseline("Hello, Rust!", Point::new(0, 16), text_style, Baseline::Top)
    .draw(&mut display)
    .expect("failed to draw text to display");
```

We're rendering "Hello, Rust!" at position (0, 16), which is 16 pixels down from the top of the screen. We use the text style we defined earlier and align the text using its top edge with Baseline::Top.

The .draw(&mut display) call renders the text into the display's internal buffer.  At this point, the text exists in RAM but is not yet visible on the physical screen.

## Displaying Text

Finally, we send the buffer contents to the actual OLED hardware:

```rust
display
    .flush()
    .await
    .expect("failed to flush data to display");
```

This is when the I2C communication happens. The driver sends the bytes from RAM to the display controller, and you'll see "Hello, Rust!" appear on your OLED screen!

## Complete Code

Here's everything put together:

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

// Interrupt Binding
use embassy_rp::peripherals::I2C0;
use embassy_rp::{bind_interrupts, i2c};

// I2C
use embassy_rp::i2c::{Config as I2cConfig, I2c};

// OLED
use ssd1306::{I2CDisplayInterface, Ssd1306Async, prelude::*};

// Embedded Graphics
use embedded_graphics::{
    mono_font::{MonoTextStyleBuilder, ascii::FONT_6X10},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
    text::{Baseline, Text},
};

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

bind_interrupts!(struct Irqs {
    I2C0_IRQ => i2c::InterruptHandler<I2C0>;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());

    let sda = p.PIN_16;
    let scl = p.PIN_17;

    let mut i2c_config = I2cConfig::default();
    i2c_config.frequency = 400_000; //400kHz

    let i2c_bus = I2c::new_async(p.I2C0, scl, sda, Irqs, i2c_config);

    let i2c_interface = I2CDisplayInterface::new(i2c_bus);

    let mut display = Ssd1306Async::new(i2c_interface, DisplaySize128x64, DisplayRotation::Rotate0)
        .into_buffered_graphics_mode();

    display
        .init()
        .await
        .expect("failed to initialize the display");

    let text_style = MonoTextStyleBuilder::new()
        .font(&FONT_6X10)
        .text_color(BinaryColor::On)
        .build();

    defmt::info!("sending text to display");
    Text::with_baseline("Hello, Rust!", Point::new(0, 16), text_style, Baseline::Top)
        .draw(&mut display)
        .expect("failed to draw text to display");

    display
        .flush()
        .await
        .expect("failed to flush data to display");

    loop {
        Timer::after_millis(100).await;
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"hello-oled"),
    embassy_rp::binary_info::rp_program_description!(c"Hello OLED"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```

## Clone the existing project

You can clone (or refer) project I created and navigate to the `hello-oled` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/oled/hello-oled
```

