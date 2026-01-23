# "Hello, Rust!" in LCD Display

We will create a simple program that prints "Hello, Rust!" on the LCD screen. This helps us quickly check that the wiring, I2C setup, and LCD configuration are correct before moving on to the next exercise.

## HD44780 Drivers

You can find driver crates by searching for the hardware controller name HD44780. Sometimes searching by the display module name, such as lcd1602, also works.

While looking around, I came across several Rust crates that can control this LCD. Some of them even support async. You could also write your own driver by referring to the datasheet, but that is beyond the scope of this chapter.

> [!TIP]
> If you want to learn how to write your own embedded Rust drivers, you can refer to the [Rust Embedded Drivers (RED) book](https://red.implrust.com/)

For now, we will use one of the existing crates. You are free to try other crates later. Just read the crate documentation and adapt the code if needed.

In this exercise, we will use this crate: [hd44780-driver](https://crates.io/crates/hd44780-driver)

### Project from template

We will start by creating a new project using the template.

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

When prompted, give your project a name, like "hello-lcd" and select `embassy` as the HAL.

### Additional Crates required

Add the following dependency to Cargo.toml along with the existing ones:

```rust
hd44780-driver = "0.4.0"
```

## Additional imports

Add the imports required for I2C and the LCD driver.

```rust
// I2C
use embassy_rp::i2c::Config as I2cConfig;
use embassy_rp::i2c::{self}; // for convenience, importing as alias

// LCD Driver
use hd44780_driver::HD44780;

use embassy_time::Delay;
```

## I2C Address

LCD1602 I2C adapters typically use address `0x27`, though some modules use `0x3F` instead depending on the adapter. Check your module's datasheet or try both addresses if you're unsure.

```rust
const LCD_I2C_ADDRESS: u8 = 0x27;
```

## I2C Setup

We'll configure the I2C interface using GPIO 16 for SDA and GPIO 17 for SCL, with a frequency of 100 kHz.

```rust
let sda = p.PIN_16;
let scl = p.PIN_17;

let mut i2c_config = I2cConfig::default();
i2c_config.frequency = 100_000; //100kHz

let i2c = i2c::I2c::new_blocking(p.I2C0, scl, sda, i2c_config);
```

## LCD Initialization

Now let's create the LCD driver instance with our I2C interface:

```rust
// LCD Init
let mut lcd =
    HD44780::new_i2c(i2c, LCD_I2C_ADDRESS, &mut Delay).expect("failed to initialize lcd");

```

## Clear the Display

Before we write anything, we'll reset and clear the screen:

```rust
// Clear the screen
lcd.reset(&mut Delay).expect("failed to reset lcd screen");
lcd.clear(&mut Delay).expect("failed to clear the screen");
```

## Write Text to the LCD

Finally, let's write our message to the LCD:

```rust
// Write to the top line
lcd.write_str("Hello, Rust!", &mut Delay)
    .expect("failed to write text to LCD");
```

## Clone the existing project

You can clone (or refer) project I created and navigate to the `hello-lcd` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/lcd/hello-lcd/
```

## rp-hal version

You can clone (or refer) project I created and navigate to the `hello-lcd` folder.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-rp-projects/lcd/hello-lcd/
```
