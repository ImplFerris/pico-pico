# Compiling for Microcontroller

Now let's talk about embedded systems. When it comes to compiling Rust code for a microcontroller, things work a little differently from normal desktop systems. Microcontrollers don’t usually run a full operating system like Linux or Windows. Instead, they run in a minimal environment, often with no OS at all. This is called a bare-metal environment.

Rust supports this kind of setup through its **no_std** mode. In normal Rust programs, the standard library (`std`) handles things like file systems, threads, heap allocation, and I/O - but none of those exist on a bare-metal microcontroller. So instead of std, we use a much smaller `core` library, which provides only the essential building blocks.

## The Target Triple for Pico 2

The Raspberry Pi Pico 2 (RP2350 chip), as you already know that it is unique; it contains selectable ARM Cortex-M33 and Hazard3 RISC-V cores . You can choose which processor architecture to use.

### ARM Cortex-M33 Target

For ARM mode, we have to use the target [`thumbv8m.main-none-eabi`](https://doc.rust-lang.org/rustc/platform-support/thumbv8m.main-none-eabi.html):

Let's break this down:

- **Architecture (thumbv8m.main)**: The Cortex-M33 uses the ARM Thumb-2 instruction set for ARMv8-M architecture.
- **Vendor (none)**: No specific vendor designation.
- **OS (none)**: No operating system - it's bare-metal.
- **ABI (eabi)**: Embedded Application Binary Interface, the standard calling convention for embedded ARM systems.

To install and use this target:
```bash
rustup target add thumbv8m.main-none-eabi
cargo build --target thumbv8m.main-none-eabi
```

### RISC-V Hazard3 Target

For RISC-V mode, use the target [`riscv32imac-unknown-none-elf`](https://doc.rust-lang.org/rustc/platform-support/riscv32-unknown-none-elf.html)`:
```sh
riscv32imac-unknown-none-elf
```

Let's break this down:

- **Architecture (riscv32imac)**: 32-bit RISC-V with I (integer), M (multiply/divide), A (atomic), and C (compressed) instruction sets.
- **Vendor (unknown)**: No specific vendor.
- **OS (none)**: No operating system - it's bare-metal.
- **Format (elf)**: ELF (Executable and Linkable Format), the object file format commonly used in embedded systems.

To install and use this target:
```bash
rustup target add riscv32imac-unknown-none-elf
cargo build --target riscv32imac-unknown-none-elf
```

In our exercises, we'll mostly use the ARM mode. Some crates like `panic-probe` don't work in RISC-V mode.

## Cargo Config

In the quick start, you might have noticed that we never manually passed the `--target` flag when running the cargo command. So how did it know which target to build for? That's because the target was already configured in the `.cargo/config.toml` file.

This file lets you store cargo-related settings, including which target to use by default. To set it up for Pico 2 in ARM mode, create a `.cargo` folder in your project root and add a `config.toml` file with the following content:

```toml
[build]
target = "thumbv8m.main-none-eabihf"
```

Now you don't have to pass `--target` every time. Cargo will use this automatically.
