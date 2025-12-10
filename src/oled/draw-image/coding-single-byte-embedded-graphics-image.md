## Drawing a Single Byte Image in Embedded Rust using embedded-graphics

By now, i hope you understand how the image is represented in the byte array. Now, let's move on to the coding part.


## Create Project

As usual, generate the project from the template with cargo-generate:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```

When prompted, give your project a name like "byte-oled" and choose "embassy" as the HAL. Enable defmt logging, if you have a debug probe so you can view logs also.


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
    image::{Image, ImageRaw},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
};
```

## Boilerplate: Initialize I2C and Display instance

We have already explained this part in the previous chapter.

```rust
let sda = p.PIN_18;
let scl = p.PIN_19;

let mut i2c_config = Config::default();
i2c_config.frequency = 400_000; //400kHz

let i2c = i2c::I2c::new_blocking(p.I2C1, scl, sda, i2c_config);

let interface = I2CDisplayInterface::new(i2c);

let mut display = Ssd1306::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
    .into_buffered_graphics_mode();

display.init().expect("failed to initialize the display");
```

## Create Your Image

We store our image as a byte array. Each byte represents one row of pixels.

```rust
// 8x5 pixels
#[rustfmt::skip]
const IMG_DATA: &[u8] = &[
    0b00111000,
    0b01000100,
    0b01000100,
    0b00101000,
    0b11101110,
];
```

This creates an Ohm symbol (Ω) that's 8 pixels wide and 5 pixels tall. Each 0b means we're writing in binary using 1s and 0s. A 1 means the pixel is ON (white), and a 0 means the pixel is OFF (black). Each line represents one row of the image from top to bottom.

## Draw the Image

Now let's put the image on the display:

```rust
let raw_image = ImageRaw::<BinaryColor>::new(IMG_DATA, 8);
let image = Image::new(&raw_image, Point::zero());
```

The first line creates a raw image from our byte data. We tell it the image is 8 pixels wide, and it figures out the height by itself. The second line places the image at position (0, 0), which is the top-left corner of the screen.

## Display the Image

Just like in the previous chapter, we need to draw the image to the display buffer and then send it to the screen:

```rust
image.draw(&mut display).expect("failed to draw text to display");
display.flush().expect("failed to flush data to display");
```

## Clone the existing project

You can also clone (or refer) project I created and navigate to the `blocking/byte-oled` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/blocking/byte-oled
```

## The Complete Code

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
    image::{Image, ImageRaw},
    pixelcolor::BinaryColor,
    prelude::Point,
    prelude::*,
};

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

// 8x5 pixels
#[rustfmt::skip]
const IMG_DATA: &[u8] = &[
    0b00111000,
    0b01000100,
    0b01000100,
    0b00101000,
    0b11101110,
];

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

    let raw_image = ImageRaw::<BinaryColor>::new(IMG_DATA, 8);

    let image = Image::new(&raw_image, Point::zero());

    image
        .draw(&mut display)
        .expect("failed to draw text to display");

    display.flush().expect("failed to flush data to display");

    loop {
        Timer::after_millis(100).await;
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"byte-oled"),
    embassy_rp::binary_info::rp_program_description!(c"your program description"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```
