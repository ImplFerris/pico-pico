# Hello Rust on OLED

The same hello world we will do with rp-hal also.

## Generating From template

To generate the project, run:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```

When prompted, choose a name for your project-let's go with "oh-led". Don't forget to select `rp-hal` as the HAL.

Then, navigate into the project folder:

```sh
cd PROJECT_NAME
# For example, if you named your project "oh-led":
# cd oh-led
```

### Add Additional Dependencies

Since we are using the SSD1306 OLED display, we need to include the SSD1306 driver. To add this dependency, use the following Cargo command:

```sh
cargo add ssd1306@0.10.0
```

We will use the embedded_graphics crate to handle graphical rendering on the OLED display, to draw images, shapes, and text.

```sh
cargo add embedded-graphics@0.8.1
```

## Additional imports

In addition to the imports from the template, you'll need the following additional dependencies for this task.

```rust
// Embedded Graphics
use embedded_graphics::mono_font::MonoTextStyleBuilder;
use embedded_graphics::mono_font::ascii::FONT_6X10;
use embedded_graphics::pixelcolor::BinaryColor;
use embedded_graphics::prelude::*;
use embedded_graphics::text::{Baseline, Text};

// For setting the Frequency
use hal::fugit::RateExtU32;
use hal::gpio::{FunctionI2C, Pin};

// SSD1306 Display
use ssd1306::{I2CDisplayInterface, Ssd1306, prelude::*};
```

## Pin Configuration

We start by configuring the GPIO pins for the I2C communication. In this case, GPIO18 is set as the SDA pin, and GPIO19 is set as the SCL pin. We then configure the I2C peripheral to work in controller mode.

```rust
// Configure two pins as being I²C, not GPIO
let sda_pin: Pin<_, FunctionI2C, _> = pins.gpio16.reconfigure();
let scl_pin: Pin<_, FunctionI2C, _> = pins.gpio17.reconfigure();

// Create the I²C drive, using the two pre-configured pins. This will fail
// at compile time if the pins are in the wrong mode, or if this I²C
// peripheral isn't available on these pins!
let i2c = hal::I2C::i2c0(
    pac.I2C0,
    sda_pin,
    scl_pin,
    400.kHz(),
    &mut pac.RESETS,
    &clocks.system_clock,
);
```

### Prepare Display

We create an interface for the OLED display using the I2C.

```rust
//helper struct is provided by the ssd1306 crate
let interface = I2CDisplayInterface::new(i2c);
// initialize the display
let mut display = Ssd1306::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
    .into_buffered_graphics_mode();
display.init().expect("failed to initialize the display");
```

### Set Text Style and Draw

Next, we define the text style and use it to display "Hello Rust" on the screen:

```rust
// Embedded graphics
let text_style = MonoTextStyleBuilder::new()
    .font(&FONT_6X10)
    .text_color(BinaryColor::On)
    .build();

Text::with_baseline(
    "Hello, Rusty!",
    Point::new(0, 16),
    text_style,
    Baseline::Top,
)
.draw(&mut display)
.expect("failed to draw text to display");
```

Here, we are writing the message at coordinates (x=0, y=16).

### Write out data to a display

```rust
display.flush().expect("failed to flush data to display");
```

## Complete code

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

// Embedded Graphics
use embedded_graphics::mono_font::MonoTextStyleBuilder;
use embedded_graphics::mono_font::ascii::FONT_6X10;
use embedded_graphics::pixelcolor::BinaryColor;
use embedded_graphics::prelude::*;
use embedded_graphics::text::{Baseline, Text};

// For setting the Frequency
use hal::fugit::RateExtU32;
use hal::gpio::{FunctionI2C, Pin};

// SSD1306 Display
use ssd1306::{I2CDisplayInterface, Ssd1306, prelude::*};

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

    // Configure two pins as being I²C, not GPIO
    let sda_pin: Pin<_, FunctionI2C, _> = pins.gpio18.reconfigure();
    let scl_pin: Pin<_, FunctionI2C, _> = pins.gpio19.reconfigure();

    // Create the I²C drive, using the two pre-configured pins. This will fail
    // at compile time if the pins are in the wrong mode, or if this I²C
    // peripheral isn't available on these pins!
    let i2c = hal::I2C::i2c1(
        pac.I2C1,
        sda_pin,
        scl_pin,
        400.kHz(),
        &mut pac.RESETS,
        &clocks.system_clock,
    );

    //helper struct is provided by the ssd1306 crate
    let interface = I2CDisplayInterface::new(i2c);
    // initialize the display
    let mut display = Ssd1306::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
        .into_buffered_graphics_mode();
    display.init().expect("failed to initialize the display");

    // Embedded graphics
    let text_style = MonoTextStyleBuilder::new()
        .font(&FONT_6X10)
        .text_color(BinaryColor::On)
        .build();

    Text::with_baseline(
        "Hello, Rusty!",
        Point::new(0, 16),
        text_style,
        Baseline::Top,
    )
    .draw(&mut display)
    .expect("failed to draw text to display");

    display.flush().expect("failed to flush data to display");

    loop {
        timer.delay_ms(100);
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

// End of file
```

## Clone the existing project

You can clone (or refer) project I created and navigate to the `hello-oled` folder.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-projects/hello-oled
```
