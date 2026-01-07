# Setup

## Picotool
picotool is a tool for working with RP2040/RP2350 binaries, and interacting with RP2040/RP2350 devices when they are in BOOTSEL mode.

[Picotool Repo](https://github.com/raspberrypi/picotool)

> [!TIP]
> Alternatively, you can download the pre-built binaries of the SDK tools from <a href="https://github.com/raspberrypi/pico-sdk-tools">here</a>, which is a simpler option than following these steps.


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
cmake -DPICO_SDK_PATH=/MY_PATH/embedded/pico-sdk/ ../
make -j8
sudo make install
```

On Linux you can add udev rules in order to run picotool without sudo:
```sh
cd ../
# In picotool cloned directory
sudo cp udev/60-picotool.rules /etc/udev/rules.d/
```


## Rust Targets
To build and deploy Rust code for the RP2350 chip, you'll need to add the appropriate targets:

```sh
rustup target add thumbv8m.main-none-eabihf
rustup target add riscv32imac-unknown-none-elf
```

## probe-rs - Flashing and Debugging Tool

probe-rs is a modern, Rust-native toolchain for flashing and debugging embedded devices. It supports ARM and RISC-V targets and works directly with hardware debug probes. When you use a Debug Probe with the Pico 2, probe-rs is the tool you rely on for both flashing firmware and debugging.

Install probe-rs using the official installer script:

```bash
curl -LsSf https://github.com/probe-rs/probe-rs/releases/latest/download/probe-rs-tools-installer.sh | sh
```

For latest installation instructions, better refer to the [official probe-rs documentation](https://probe.rs/).

By default, debug probes on Linux can only be accessed with root privileges. To avoid using sudo for every command, you should install the appropriate udev rules that allow regular users to access the probe. Follow [the instructions provided](https://probe.rs/docs/getting-started/probe-setup/).

**Quick summary:**
1. [Download the udev rules](https://probe.rs/files/69-probe-rs.rules) file from the probe-rs repository
2. Copy it to `/etc/udev/rules.d/`
3. Reload udev rules with `sudo udevadm control --reload`
4. Unplug and replug your Debug Probe

After this setup, you can use probe-rs without root privileges.
