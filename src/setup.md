# Setup

## Picotool
picotool is a tool for working with RP2040/RP2350 binaries, and interacting with RP2040/RP2350 devices when they are in BOOTSEL mode.

[Picotool Repo](https://github.com/raspberrypi/picotool)

<div class="alert-box alert-box-info">
    <span class="icon"><i class="fa fa-info"></i></span>
    <div class="alert-content">
        <b class="alert-title">Pre-built binaries</b>
        <p>Alternatively, you can download the pre-built binaries of the SDK tools from <a href="https://github.com/raspberrypi/pico-sdk-tools">here</a>, which is a simpler option than following these steps.</p>
    </div>
</div>


Here's a quick summary of the steps I followed:
```sh
# Install dependencies
sudo apt install build-essential pkg-config libusb-1.0-0-dev cmake

mkdir embedded && cd embedded

# Clone the Pico SDK
git clone https://github.com/raspberrypi/pico-sdk
cd pico-sdk
git submodule update --init lib/mbedtls
cd ../

# Set the environment variable for the Pico SDK
PICO_SDK_PATH=/MY_PATH/embedded/pico-sdk

# Clone the Picotool repository
git clone https://github.com/raspberrypi/picotool
```

Build and install Picotool
```sh
cd picotool
mkdir build && cd build
# cmake ../
cmake -DPICO_SDK_PATH=/var/ws/embedded/pico-sdk/ ../
make -j8
sudo make install
```

On Linux you can add udev rules in order to run picotool without sudo:
```sh
cd ../
# In picotool cloned directory
sudo cp udev/99-picotool.rules /etc/udev/rules.d/
```


## Rust Targets
To build and deploy Rust code for the RP2350 chip, you'll need to add the appropriate targets:

```sh
rustup target add thumbv8m.main-none-eabihf
rustup target add riscv32imac-unknown-none-elf
```
