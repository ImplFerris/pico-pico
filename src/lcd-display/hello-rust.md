# "Hello, Rust!" in LCD Display

In this program, we will just print "Hello, Rust!" text in the LCD display. 

## HD44780 Drivers
During my research, I came across many Rust crates for controll the LCD Display, but these two stood out as working well. In this program, we will start by using the `hd44780-driver` crate.
- [hd44780-driver](https://crates.io/crates/hd44780-driver) 
- [liquid_crystal](https://crates.io/crates/liquid_crystal) 


### Project from template

To set up the project, run:
```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git
```
When prompted, give your project a name, like "lcd-hello" and select `RP-HAL` as the HAL.

Then, navigate into the project folder:
```sh
cd PROJECT_NAME
# For example, if you named your project "lcd-hello":
# cd lcd-hello
```

### Additional Crates required
Update your Cargo.toml to add these additional crate along with the existing dependencies.

```rust
hd44780-driver = "0.4.0"
```

 ### Additional imports

```rust
use hd44780_driver::HD44780;
```


### Mapping Pico and LCD Pins

We connect GPIO16 to the RS pin, GPIO17 to the Enable (E) pin, and GPIO18-21 to the D4-D7 data pins of the LCD. We're using only 4 data pins since we will be working on 4-bit mode.

```rust
// Read Select Pin
let rs = pins.gpio16.into_push_pull_output();

// Enable Pin
let en = pins.gpio17.into_push_pull_output();

// Data Pins
let d4 = pins.gpio18.into_push_pull_output();
let d5 = pins.gpio19.into_push_pull_output();
let d6 = pins.gpio20.into_push_pull_output();
let d7 = pins.gpio21.into_push_pull_output();

```

### Write Text to the LCD
Here, we initialize the LCD module, clear the screen, and then write the text "Hello, Rust!".
```rust
// LCD Init
let mut lcd = HD44780::new_4bit(rs, en, d4, d5, d6, d7, &mut timer).unwrap();

// Clear the screen
lcd.reset(&mut timer).unwrap();
lcd.clear(&mut timer).unwrap();

// Write to the top line
lcd.write_str("Hello, Rust!", &mut timer).unwrap();
```

## Clone the existing project
You can clone (or refer) project I created and navigate to the `lcd-hello` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/lcd-hello/
```
