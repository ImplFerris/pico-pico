# Setup

## Picotool
picotool is a tool for working with RP2040/RP2350 binaries, and interacting with RP2040/RP2350 devices when they are in BOOTSEL mode.

[Picotool Repo](https://github.com/raspberrypi/picotool)

Here’s a quick summary of the steps I followed:
```sh
mkdir embedded && cd embedded

# Clone the Pico SDK
git clone https://github.com/raspberrypi/pico-sdk
# Set the environment variable for the Pico SDK
PICO_SDK_PATH=/MY_PATH/embedded/pico-sdk

# Clone the Picotool repository
git clone https://github.com/raspberrypi/picotool
```

Build and install Picotool
```sh
cd picotool
cmake ../
make -j8
sudo make install
```

On Linux you can add udev rules in order to run picotool without sudo:
```sh
sudo cp udev/99-picotool.rules /etc/udev/rules.d/
```


## Rust Targets
To build and deploy Rust code for the RP2350 chip, you'll need to add the appropriate targets:

```sh
rustup target add thumbv8m.main-none-eabihf
rustup target add riscv32imac-unknown-none-elf
```
