# Pico to PC

The example [provided](https://github.com/rp-rs/rp-hal/blob/main/rp235x-hal-examples/src/bin/usb.rs) in the RP-HAL repository sends a simple "Hello, World!" message from the Pico to the computer once the timer ticks reach 2,000,000. To ensure the message is only sent once, we add a check that sends it only on the first occurrence. Also, it polls for any incoming data to the device (Pico). If data is received, it converts it to uppercase and send it back(This is just show communication is working, not just echoing). 

We'll slightly modify the code to make it more fun. Instead of sending "Hello, World!", we'll send "Hello, Rust!" to the computer. Wait, I know that's not the fun part. Here it comes: if you type 'r' in the terminal connected via USB serial, the onboard LED will turn on. Type anything else, and the LED will turn off.


## Project from template

To set up the project, run:
```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git
```
When prompted, give your project a name, like "usb-fun" and select `RP-HAL` as the HAL.

Then, navigate into the project folder:
```sh
cd PROJECT_NAME
# For example, if you named your project "usb-fun":
# cd usb-fun
```

### Additional Crates required
Update your Cargo.toml to add these additional crate along with the existing dependencies.
```rust
usbd-serial = "0.2.2"
usb-device = "0.3.2"
```

### Additional imports
```rust
// USB Device support
use usb_device::{class_prelude::*, prelude::*};
// USB Communications Class Device support
use usbd_serial::SerialPort;
```

## Set up the USB driver
```rust
let usb_bus = UsbBusAllocator::new(hal::usb::UsbBus::new(
    pac.USB,
    pac.USB_DPRAM,
    clocks.usb_clock,
    true,
    &mut pac.RESETS,
));
```

## Set up the USB Communications Class Device driver
```rust
let mut serial = SerialPort::new(&usb_bus);
```

## Create a USB device with a fake VID and PID

```rust
let mut usb_dev = UsbDeviceBuilder::new(&usb_bus, UsbVidPid(0x16c0, 0x27dd))
    .strings(&[StringDescriptors::default()
        .manufacturer("implRust")
        .product("Ferris")
        .serial_number("TEST")])
    .unwrap()
    .device_class(2) // 2 for the CDC, from: https://www.usb.org/defined-class-codes
    .build();
```

## Sending Message to PC

This part sends "Hello, Rust!" to the PC when the timer count exceeds 2,000,000 by writing the text to the serial port. We ensure the message is sent only once. 
```rust
if !said_hello && timer.get_counter().ticks() >= 2_000_000 {
    said_hello = true;
    // Writes bytes from `data` into the port and returns the number of bytes written.
    let _ = serial.write(b"Hello, Rust!\r\n");
}
```

## Polling for data 

Here is the fun part. When you type characters on your computer, they are sent to the Pico via USB serial. On the Pico, we check if the received character matches the letter 'r'. If it matches, the onboard LED turns on. For any other character, the LED turns off.

```rust
if usb_dev.poll(&mut [&mut serial]) {
    let mut buf = [0u8; 64];
    if let Ok(count) = serial.read(&mut buf) {
        for &byte in &buf[..count] {
            if byte == b'r' {
                led.set_high().unwrap();
            } else {
                led.set_low().unwrap();
            }
        }
    }
}
```

## The Full code

```rust
#![no_std]
#![no_main]

use embedded_hal::digital::OutputPin;
use hal::block::ImageDef;
use panic_halt as _;
use rp235x_hal as hal;

use usb_device::{class_prelude::*, prelude::*};
use usbd_serial::SerialPort;

#[link_section = ".start_block"]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

const XTAL_FREQ_HZ: u32 = 12_000_000u32;

#[hal::entry]
fn main() -> ! {
    let mut pac = hal::pac::Peripherals::take().unwrap();
    let mut watchdog = hal::Watchdog::new(pac.WATCHDOG);

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
    let timer = hal::Timer::new_timer0(pac.TIMER0, &mut pac.RESETS, &clocks);

    let sio = hal::Sio::new(pac.SIO);
    let pins = hal::gpio::Pins::new(
        pac.IO_BANK0,
        pac.PADS_BANK0,
        sio.gpio_bank0,
        &mut pac.RESETS,
    );
    let mut led = pins.gpio25.into_push_pull_output();

    let usb_bus = UsbBusAllocator::new(hal::usb::UsbBus::new(
        pac.USB,
        pac.USB_DPRAM,
        clocks.usb_clock,
        true,
        &mut pac.RESETS,
    ));

    let mut serial = SerialPort::new(&usb_bus);

    let mut usb_dev = UsbDeviceBuilder::new(&usb_bus, UsbVidPid(0x16c0, 0x27dd))
        .strings(&[StringDescriptors::default()
            .manufacturer("implRust")
            .product("Ferris")
            .serial_number("TEST")])
        .unwrap()
        .device_class(2) // 2 for the CDC, from: https://www.usb.org/defined-class-codes
        .build();

    let mut said_hello = false;
    loop {
        // Send data to the PC
        if !said_hello && timer.get_counter().ticks() >= 2_000_000 {
            said_hello = true;
            // Writes bytes from `data` into the port and returns the number of bytes written.
            let _ = serial.write(b"Hello, Rust!\r\n");
        }

        // Read data from PC
        if usb_dev.poll(&mut [&mut serial]) {
            let mut buf = [0u8; 64];
            if let Ok(count) = serial.read(&mut buf) {
                for &byte in &buf[..count] {
                    if byte == b'r' {
                        led.set_high().unwrap();
                    } else {
                        led.set_low().unwrap();
                    }
                }
            }
        }
    }
}

#[link_section = ".bi_entries"]
#[used]
pub static PICOTOOL_ENTRIES: [hal::binary_info::EntryAddr; 5] = [
    hal::binary_info::rp_cargo_bin_name!(),
    hal::binary_info::rp_cargo_version!(),
    hal::binary_info::rp_program_description!(c"USB Fun"),
    hal::binary_info::rp_cargo_homepage_url!(),
    hal::binary_info::rp_program_build_attribute!(),
];

```


## Clone the existing project
You can clone (or refer) project I created and navigate to the `usb-fun` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/usb-fun/
```

## How to Run ?
The method to flash (run the code) on the Pico is the same as usual. However, we need to set up tio to interact with the Pico through the serial port (/dev/ttyACM0). This allows us to read data from the Pico or send data to it.

### tio
Make sure you have tio installed on your system. If not, you can install it using:
```sh
apt install tio
```

### Connecting to the Serial Port
Run the following command to connect to the Pico's serial port:

```sh
tio /dev/ttyACM0
```
This will open a terminal session for communicating with the Pico.

### Flashing and Running the Code
Open another terminal, navigate to the project folder, and flash the code onto the Pico as usual:
```sh
cargo run
```
If everything is set up correctly, you should see a "Connected" message in the tio terminal, followed by the "Hello, Rust!" message sent from the Pico.

<img style="display: block; margin: auto;" src="./images/tio-usb-pico.png"/>


### Send data to Pico
In the terminal where tio is running, you type that will be sent to the Pico. You won't see what you type (since we're not echoing back the input). 

If you press the letter 'r', the onboard LED will be turned on. If you press any other character, the LED will be turned off. 
