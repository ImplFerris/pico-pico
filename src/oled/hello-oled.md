# Hello Rust on OLED

We will create a simple program to display "Hello, Rust" in the OLED display.

## Generating From template

Refer to the [Template section](../cargo-generate.md) for details and instructions.

To generate the project, run:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git
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
cargo add ssd1306@0.9.0
```

We will use the embedded_graphics crate to handle graphical rendering on the OLED display, to draw images, shapes, and text.
```sh
cargo add embedded-graphics@0.8.1
```

## Code

### Additional imports
In addition to the imports from the template, you'll need the following additional dependencies for this task.
```rust
use hal::fugit::RateExtU32;
use hal::gpio::{FunctionI2C, Pin};
use ssd1306::{prelude::*, I2CDisplayInterface, Ssd1306};
use embedded_graphics::prelude::*;
use embedded_graphics::mono_font::ascii::FONT_6X10;
use embedded_graphics::mono_font::MonoTextStyleBuilder;
use embedded_graphics::pixelcolor::BinaryColor;
use embedded_graphics::text::{Baseline, Text};
```

### Pin Configuration
We start by configuring the GPIO pins for the I2C communication. In this case, GPIO18 is set as the SDA pin, and GPIO19 is set as the SCL pin. We then configure the I2C peripheral to work in master mode.

```rust
// Configure two pins as being I²C, not GPIO
let sda_pin: Pin<_, FunctionI2C, _> = pins.gpio18.reconfigure();
let scl_pin: Pin<_, FunctionI2C, _> = pins.gpio19.reconfigure();

let i2c = hal::I2C::i2c1(
    pac.I2C1,
    sda_pin,i2c1
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
display.init().unwrap();
```

### Set Text Style and Draw
Next, we define the text style and use it to display "Hello Rust" on the screen:
```rust
// Embedded graphics 
let text_style = MonoTextStyleBuilder::new()
    .font(&FONT_6X10)
    .text_color(BinaryColor::On)
    .build();

Text::with_baseline("Hello, Rust!", Point::new(0, 16), text_style, Baseline::Top)
    .draw(&mut display)
    .unwrap();
```

### Write out data to a display
```rust
display.flush().unwrap();
``` 

## Full logic
```rust
let sda_pin: Pin<_, FunctionI2C, _> = pins.gpio18.reconfigure();
let scl_pin: Pin<_, FunctionI2C, _> = pins.gpio19.reconfigure();

let i2c = hal::I2C::i2c1(
    pac.I2C1,
    sda_pin,
    scl_pin,
    400.kHz(),
    &mut pac.RESETS,
    &clocks.system_clock,
);

let interface = I2CDisplayInterface::new(i2c);

let mut display = Ssd1306::new(interface, DisplaySize128x64, DisplayRotation::Rotate0)
    .into_buffered_graphics_mode();

display.init().unwrap();
let text_style = MonoTextStyleBuilder::new()
    .font(&FONT_6X10)
    .text_color(BinaryColor::On)
    .build();

Text::with_baseline("Hello, Rust!", Point::new(0, 16), text_style, Baseline::Top)
    .draw(&mut display)
    .unwrap();

display.flush().unwrap();
loop {
    timer.delay_ms(500);
}
```

## Clone the existing project
You can clone (or refer) project I created and navigate to the `hello-oled` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/hello-oled
```
