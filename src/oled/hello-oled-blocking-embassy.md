# Hello OLED

We are going to keep things simple. We are just going to display "Hello, Rust!" on the OLED display. We will use Embassy but with I2C in blocking mode. Wait, you might wonder why blocking mode, especially with Embassy?

There are situations where you will need blocking mode even in async projects. But the main reason we are choosing it here is because we have not introduced the interrupts concept yet in this book. So, we will start without it. 


## Create Project

As usual, generate the project from the template with cargo-generate:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```

When prompted, give your project a name like "hello-oled" and choose "embassy" as the HAL. Enable defmt logging, if you have a debug probe so you can view logs also.

## Update Dependencies

Add the following lines to your Cargo.toml under dependencies:

```toml
embedded-graphics = "0.8.1"
ssd1306 = "0.10.0"
```

## Additional imports

Add these imports at the top of your main.rs:

```rust
// I2C
use embassy_rp::i2c::{self, Config};

// OLED
use ssd1306::{I2CDisplayInterface, Ssd1306, prelude::*};

// Embedded Graphics
use embedded_graphics::{
    mono_font::{MonoTextStyleBuilder, ascii::FONT_6X10},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
    text::{Baseline, Text},
};
```

The embassy_rp::i2c import provides I2C functionality for the RP2350. The ssd1306 import gives us the display driver we discussed earlier. Finally, embedded_graphics provides text rendering capabilities.

## Initialize I2C

First, we need to set up the I2C bus to communicate with the display.

```rust
let sda = p.PIN_18;
let scl = p.PIN_19;

let mut i2c_config = Config::default();
i2c_config.frequency = 400_000; //400kHz

let i2c = i2c::I2c::new_blocking(p.I2C1, scl, sda, i2c_config);
```

We have connected the OLED's SDA line to Pin 18 and the SCL line to Pin 19. Throughout this chapter we will keep using these same pins. If you have connected your display to a different valid I2C pair, then adjust the code to match your wiring.

We are using the "new_blocking" method to create an I2C instance in blocking mode. You can use the default I2C configuration which runs at 100 kHz, or change it to 400 kHz which is the maximum frequency the OLED safely supports.

## Initialize Display

Now we create the display interface and initialize it:

```rust
let interface = I2CDisplayInterface::new(i2c);

let mut display = Ssd1306::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
    .into_buffered_graphics_mode();
```

```rust
display.init().expect("failed to initialize the display");
```

The I2CDisplayInterface::new(i2c) is a helper struct that wraps our I2C bus and prepares it for the SSD1306. It automatically uses the default I2C address 0x3C, which is standard for most SSD1306 displays.

When we call Ssd1306::new(...), we create a new display instance. We pass it the interface we just created, specify that our display is 128 pixels wide by 64 pixels tall with DisplaySize128x64, and set DisplayRotation::Rotate0 for normal orientation (no rotation).

The .into_buffered_graphics_mode() method converts the display into buffered graphics mode. This gives us the DrawTarget trait so we can use embedded-graphics for drawing.

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

// I2C
use embassy_rp::i2c::{self, Config};

// OLED
use ssd1306::{I2CDisplayInterface, Ssd1306, prelude::*};

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

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());

    let sda = p.PIN_18;
    let scl = p.PIN_19;

    let mut i2c_config = Config::default();
    i2c_config.frequency = 400_000; //400kHz

    let i2c = i2c::I2c::new_blocking(p.I2C1, scl, sda, i2c_config);

    let interface = I2CDisplayInterface::new(i2c);

    let mut display = Ssd1306::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
        .into_buffered_graphics_mode();

    display.init().expect("failed to initialize the display");
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

You can clone (or refer) project I created and navigate to the `blocking/hello-oled` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/blocking/hello-oled
```

