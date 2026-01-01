# Embedded Rust Code to Control MAX7219 LED Matrix on Raspberry Pi Pico 2

Let us move on to the coding part. In this program, we will draw a square on the 8x8 LED matrix using embedded-graphics.

As usual, create a new project using the provided template.

```rust
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```

When prompted, give your project a name, like "hello-max7219" and select embassy as the HAL.

## Additional crates

We will add the following crates along with existing dependencies

```toml
max7219-display = { version = "0.1.5", features = ["led-matrix", "graphics"] }
embedded-graphics = "0.8.0"
embedded-hal-bus = "0.3.0"
```

The max7219-display crate provides the driver used in this program. The driver supports different kinds of display modules, such as 7-segment displays and LED matrices. In our case, we are using an 8x8 LED matrix, so we enable the led-matrix feature. Since we want to draw shapes instead of manually controlling individual LEDs, we also enable the graphics feature, which implements the embedded-graphics traits for the driver.

The embedded-graphics crate is used to create shapes and text in a structured way. It provides primitives like rectangles, points, and styles, which makes drawing on the LED matrix much easier.

### embedded-hal-bus

The embedded-hal crate defines common traits for peripherals like SPI, enabling drivers to work across different microcontrollers. To understand why embedded-hal-bus is needed, it helps to look at how these traits are organized.

Embedded HAL separates SPI into two distinct concepts: the **SPI bus** (representing the shared clock and data lines) and the **SPI device** (representing a single peripheral with its own chip select line). Microcontroller HALs typically provide an SPI bus implementation, but device drivers are written to work with SPI devices.

The embedded-hal-bus crate bridges this gap. It provides adapters that wrap an SPI bus and manage the chip select pin, creating the SPI device interface that drivers expect. In this chapter, we use `ExclusiveDevice`, the simplest adapter. It's designed for scenarios where only one device uses the SPI bus, it automatically asserts the chip select at the start of each transaction and deasserts it when complete.

## Additional Imports

We now add the imports needed for SPI, the MAX7219 display driver, and drawing with embedded-graphics. 
```rust
// For MAX7219
use embedded_hal_bus::spi::ExclusiveDevice;
use max7219_display::led_matrix::display::SingleMatrix;

// For Drawing shapes
use embedded_graphics::pixelcolor::BinaryColor;
use embedded_graphics::prelude::*;
use embedded_graphics::primitives::{PrimitiveStyleBuilder, Rectangle};

// For SPI
use embassy_rp::spi::{Config as SpiConfig, Spi};

// For CS Pin
use embassy_rp::gpio::{Level, Output};
```

## SPI setup and chip select

Let's start by creating the chip select pin. The MAX7219 uses an active-low chip select line, so we initialize the pin in the high state. This keeps the device inactive until an SPI transfer begins.

```rust
let cs_pin = Output::new(p.PIN_13, Level::High);
```

Next, we assign the SPI clock and MOSI pins. Since the MAX7219 is a write-only device, only these two signals are required. There is no MISO pin involved in this setup.

```rust
let clk = p.PIN_14;
let mosi = p.PIN_15;
```

With the pins selected, we create the SPI peripheral. We use a transmit-only blocking SPI instance since all communication goes from the Pico to the display.

```rust
let spi_bus = Spi::new_blocking_txonly(p.SPI1, clk, mosi, SpiConfig::default());
```

At this point, we have an SPI bus, but it does not yet manage chip select. To handle that, we wrap the bus using ExclusiveDevice. This turns the SPI bus into an SPI device by automatically controlling the chip select pin for each transfer.

```rust
let spi_dev =
        ExclusiveDevice::new_no_delay(spi_bus, cs_pin).expect("Failed to get exclusive device");
```

After this step, the SPI device is ready to be used by the MAX7219 display driver.

## Creating the display instance

With the SPI device ready, we now create the LED matrix display instance. Here we are working with a single 8x8 LED matrix, not a daisy-chained setup. The display driver takes ownership of the SPI device and uses it to send commands and pixel data to the MAX7219.

```rust
// Create a display instance for a single 8x8 LED matrix (not daisy-chained)
let mut display = SingleMatrix::from_spi(spi_dev).expect("display count 1 should not panic");
```

## Setting the display brightness

After creating the display instance, we set the brightness level. The MAX7219 supports multiple intensity levels, and here we configure it for the only device at index 0. A low intensity value is usually sufficient and avoids excessive brightness.

```rust
// Set brightness (intensity level) of the only device at index 0
    display
        .driver()
        .set_intensity(0, 1)
        .expect("failed to set intensity");
```


## Drawing a rectangle

Now we draw a simple shape using embedded-graphics. Instead of filling the entire area, we draw a hollow rectangle so only the border pixels are turned on. I first tried a filled rectangle, but it did not look good, so I have commented out that version.

First, we define a drawing style. This style turns pixels on and sets the stroke width to one pixel, which works well for an 8x8 display.

```rust
// ---- Draw Rectangle ----
// let rect = Rectangle::new(Point::new(1, 1), Size::new(6, 6)).into_styled(
//     embedded_graphics::primitives::PrimitiveStyle::with_fill(BinaryColor::On),
// );
let hollow_rect_style = PrimitiveStyleBuilder::new()
    .stroke_color(BinaryColor::On) // Only draw the border
    .stroke_width(1) // Border thickness of 1 pixel
    .build();
```

Next, we define the rectangle. The top-left corner is placed at x = 1 and y = 1, and the rectangle spans 6 pixels in both width and height. This leaves a one-pixel margin around the edges, so the border does not touch the outer LEDs.

```rust
let rect = Rectangle::new(Point::new(1, 1), Size::new(6, 6)).into_styled(hollow_rect_style);
```

When we draw the rectangle, it is written into the display buffer. At this stage, nothing has been sent to the hardware yet.

```rust
rect.draw(&mut display).expect("failed to draw the shape");
```

To actually update the LED matrix, we flush the framebuffer. This sends the pixel data over SPI to the MAX7219.

```rust
display.flush().expect("failed to send to the device");
```

## The full code

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_rp as hal;
use embassy_rp::block::ImageDef;
use embassy_time::Timer;

// For MAX7219
use embedded_hal_bus::spi::ExclusiveDevice;
use max7219_display::led_matrix::display::SingleMatrix;

// For Drawing shapes
use embedded_graphics::pixelcolor::BinaryColor;
use embedded_graphics::prelude::*;
use embedded_graphics::primitives::{PrimitiveStyleBuilder, Rectangle};

// For SPI
use embassy_rp::spi::{Config as SpiConfig, Spi};

// For CS Pin
use embassy_rp::gpio::{Level, Output};

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

    let cs_pin = Output::new(p.PIN_13, Level::High);

    let clk = p.PIN_14;
    let mosi = p.PIN_15;

    let spi_bus = Spi::new_blocking_txonly(p.SPI1, clk, mosi, SpiConfig::default());
    let spi_dev =
        ExclusiveDevice::new_no_delay(spi_bus, cs_pin).expect("Failed to get exclusive device");

    // Create a display instance for a single 8x8 LED matrix (not daisy-chained)
    let mut display = SingleMatrix::from_spi(spi_dev).expect("display count 1 should not panic");

    // Set brightness (intensity level) of the only device at index 0
    display
        .driver()
        .set_intensity(0, 1)
        .expect("failed to set intensity");

    // ---- Draw Rectangle ----
    // let rect = Rectangle::new(Point::new(1, 1), Size::new(6, 6)).into_styled(
    //     embedded_graphics::primitives::PrimitiveStyle::with_fill(BinaryColor::On),
    // );
    let hollow_rect_style = PrimitiveStyleBuilder::new()
        .stroke_color(BinaryColor::On) // Only draw the border
        .stroke_width(1) // Border thickness of 1 pixel
        .build();
    let rect = Rectangle::new(Point::new(1, 1), Size::new(6, 6)).into_styled(hollow_rect_style);
    rect.draw(&mut display).expect("failed to draw the shape");

    display.flush().expect("failed to send to the device");

    loop {
        Timer::after_millis(100).await;
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"hello-max7219"),
    embassy_rp::binary_info::rp_program_description!(c"your program description"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```


## Clone the existing project

You can clone (or refer to) the project I created and navigate to the `hello-max7219` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/max7219/hello-max7219
```
