# Flashing the Rust Firmware into Raspberry Pi Pico 2

After building our program, we'll have an ELF binary file ready to flash.

For a debug build (`cargo build`), you'll find the file here:

```sh
./target/thumbv8m.main-none-eabihf/debug/pico-from-scratch
```

For a release build (`cargo build --release`), you'll find it here:

```sh
./target/thumbv8m.main-none-eabihf/release/pico-from-scratch
```

To load our program onto the Pico, we'll use a tool called [Picotool](https://github.com/raspberrypi/picotool). Here's the command to flash our program:

```rust
picotool load -u -v -x -t elf ./target/thumbv8m.main-none-eabihf/debug/pico-from-scratch
```

Here's what each flag does:

- `-u` for update mode (only writes what's changed)
- `-v` to verify everything wrote correctly
- `-x` to run the program immediately after loading
- `-t elf` tells picotool we're using an ELF file

## cargo run command

Typing that long command every time gets tedious. Let's simplify it by updating the `.cargo/config.toml` file. We can configure Cargo to automatically use picotool when we run `cargo run`:

```toml
[target.thumbv8m.main-none-eabihf]
runner = "picotool load -u -v -x -t elf"
```

Now, you can just type:

```sh
cargo run --release

#or

cargo run
```

and your program will be flashed and executed on the Pico.

At this point, it still won't actually flash. We're missing one important step.
