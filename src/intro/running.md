# Running the program
Before we explore further examples, let’s cover the general steps to build and run any program on the Raspberry Pi Pico 2.

## Build and Run for ARM
```sh
# build the program
cargo build --target=thumbv8m.main-none-eabihf

# Run the program
cargo run --bin blinky --target=thumbv8m.main-none-eabihf
```


## Build and Run for RISC-V
```sh
# build the program
cargo build --target=riscv32imac-unknown-none-elf

# Run the program
cargo run --bin blinky --target=riscv32imac-unknown-none-elf
```
