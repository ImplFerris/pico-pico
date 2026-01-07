# Read SD Card with Raspberry Pi Pico

In this chapter, we are going to read a file from a microSD card and print its contents. Before continuing, you will need an SD card that is already formatted as FAT32, and it must contain a file named RUST.TXT. For this example, you can put the text Ferris inside.

## Select Your Output Method

In this section, I have given two output methods. Which one you use depends on whether you have a debug probe or not.

There is no major difference in terms of logic or behavior. The program works the same way in both cases. The main difference is how the output is printed.

If you are using a debug probe, we print messages using defmt over RTT. If you are using USB serial, we print messages using the log crate.

The USB serial setup is the same as what we used in the earlier USB Serial chapter. You may need to follow the same steps to set up the USB logger. I will not be repeating those steps here.

From a code point of view, the difference mostly comes down to this:

- Since we use defmt with a debug probe, logging is done using defmt::info!
- On the other hand, when using USB serial, we use the log crate, so logging is done with log::info!.

Select the tab below based on your setup:

{{#tabs global="log-method" }}
{{#tab name="Debug Probe" }}
In this setup, we use defmt over RTT to print the file contents.
{{#endtab }}
{{#tab name="USB Serial" }}
In this setup, we use USB serial logging to print the file contents.
{{#endtab }}
{{#endtabs }}

## Project from template

We will start by creating a new project using the template. 

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

{{#tabs global="log-method" }}
{{#tab name="Debug Probe" }}
When prompted, enter a project name, for example "read-sdcard", and select "Embassy" as the HAL. Ensure to enable "defmt" for logging.
{{#endtab }}
{{#tab name="USB Serial" }}
When prompted, enter a project name, for example "read-sdcard", and select "Embassy" as the HAL. You do not need to enable defmt for this setup.
{{#endtab }}
{{#endtabs }}

## Additional Crates required

Update your Cargo.toml to add these additional crate along with the existing dependencies.

{{#tabs global="log-method" }}
{{#tab name="Debug Probe" }}
```toml
# To convert Spi bus to SpiDevice
embedded-hal-bus = "0.3.0"

# sd card driver
embedded-sdmmc = "0.9.0"
```
{{#endtab }}
{{#tab name="USB Serial" }}
```toml
# To convert Spi bus to SpiDevice
embedded-hal-bus = "0.3.0"

# sd card driver
embedded-sdmmc = "0.9.0"

embassy-usb-logger = "0.5.1"
log = "0.4"
```
{{#endtab }}
{{#endtabs }}


The embedded-sdmmc crate is a driver for reading and writing files on FAT-formatted SD cards.

## Additional imports

```rust
// For SPI
use embassy_rp::spi;
use embassy_rp::spi::Spi;
use embassy_time::Delay;
use embedded_hal_bus::spi::ExclusiveDevice;

// For CS Pin
use embassy_rp::gpio::{Level, Output};

// For SdCard
use embedded_sdmmc::{SdCard, TimeSource, Timestamp, VolumeIdx, VolumeManager};
```


## Dummy TimeSource 

The SD card filesystem code requires a TimeSource implementation to supply timestamps for file metadata. Even though we are not creating or modifying files in this example, the trait still needs to be implemented.

Since timestamps are not important here, we use a dummy implementation that always returns zeroed values.

```rust
/// Code from https://github.com/rp-rs/rp-hal-boards/blob/main/boards/rp-pico/examples/pico_spi_sd_card.rs
/// A dummy timesource, which is mostly important for creating files.
#[derive(Default)]
pub struct DummyTimesource();

impl TimeSource for DummyTimesource {
    // In theory you could use the RTC of the rp2040 here, if you had
    // any external time synchronizing device.
    fn get_timestamp(&self) -> Timestamp {
        Timestamp {
            year_since_1970: 0,
            zero_indexed_month: 0,
            zero_indexed_day: 0,
            hours: 0,
            minutes: 0,
            seconds: 0,
        }
    }
}
```

## Setting up SPI for the SD card reader

Now we configure the SPI peripheral and the GPIO pins used to communicate with the SD card reader.

```rust
let miso = p.PIN_4;
let cs_pin = Output::new(p.PIN_5, Level::High);
let clk = p.PIN_6;
let mosi = p.PIN_7;

let mut config = spi::Config::default();
config.frequency = 400_000;

let spi_bus = Spi::new_blocking(p.SPI0, clk, mosi, miso, config);
```

## Creating an SpiDevice from the SPI bus

The SD card driver expects an SpiDevice, not a raw SPI bus.

```rust
let spi_device =
    ExclusiveDevice::new(spi_bus, cs_pin, Delay).expect("Failed to get exclusive device");
```

## Setting up the SD card driver

With the SPI device ready, we can now create the SD card driver instance.

```rust
let sdcard = SdCard::new(spi_device, Delay);
```

## Reading the SD card size

Before opening any files, we read the total size of the card to confirm that initialization succeeded.

{{#tabs global="log-method" }}
{{#tab name="Debug Probe" }}
```rust
defmt::info!("Init SD card controller and retrieve card size...");
let sd_size = sdcard.num_bytes().expect("failed to get sdcard size");
defmt::info!("card size is {} bytes", sd_size);
```
{{#endtab }}
{{#tab name="USB Serial" }}
```rust
log::info!("Init SD card controller and retrieve card size...");
let sd_size = sdcard.num_bytes().expect("failed to get sdcard size");
log::info!("card size is {} bytes", sd_size);
```
{{#endtab }}
{{#endtabs }}


## Opening the volume and root directory

Next, we create a volume manager and open the first volume on the card.

```rust
let volume_mgr = VolumeManager::new(sdcard, DummyTimesource::default());
let volume0 = volume_mgr
    .open_volume(VolumeIdx(0))
    .expect("failed to open volume");

let root_dir = volume0.open_root_dir().expect("failed to open root dir");
```

## Opening the file

With the root directory available, we open the file we want to read.

```rust
let my_file = root_dir
    .open_file_in_dir("RUST.TXT", embedded_sdmmc::Mode::ReadOnly)
    .expect("failed to open RUST.TXT file");
```

## Reading the file content

Finally, we read the file in small chunks and print its contents.


{{#tabs global="log-method" }}
{{#tab name="Debug Probe" }}
```rust
while !my_file.is_eof() {
    let mut buffer = [0u8; 32];

    if let Ok(n) = my_file.read(&mut buffer) {
        if let Ok(s) = core::str::from_utf8(&buffer[..n]) {
            defmt::info!("{}", s);
        } else {
            defmt::info!("{:02x}", &buffer[..n]);
        }
    }
}
```
{{#endtab }}
{{#tab name="USB Serial" }}
```rust
while !my_file.is_eof() {
    let mut buffer = [0u8; 32];

    if let Ok(n) = my_file.read(&mut buffer) {
        if let Ok(s) = core::str::from_utf8(&buffer[..n]) {
            log::info!("{}", s);
        } else {
            log::info!("{:02x}", &buffer[..n]);
        }
    }
}
```
{{#endtab }}
{{#endtabs }}


## Clone the existing project

{{#tabs global="log-method" }}
{{#tab name="Debug Probe" }}
You can clone (or refer) project I created and navigate to the `read-sdcard` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/sdcard/read-sdcard/
```
{{#endtab }}
{{#tab name="USB Serial" }}
You can clone (or refer) project I created and navigate to the `print-over-usb` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/sdcard/print-over-usb/
```
{{#endtab }}
{{#endtabs }}


Once the program is running, the output will show the contents of RUST.TXT file.
