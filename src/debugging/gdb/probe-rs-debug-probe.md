# Debugging Embedded Rust on Raspberry Pi Pico 2 with GDB

In this chapter, we will look at how to debug Embedded Rust programs on the Raspberry Pi Pico 2 (RP2350) using GDB. You will need a Debug Probe hardware and you must connect it to your Raspberry Pi Pico 2. Make sure you have read [chapter "Debug Probe for Raspberry Pi Pico 2"](../pico-debug-probe.md) before continuing.

## What a Debug Probe Gives You

In the Debug Probe introduction chapter, we saw that it helps you avoid pressing the BOOTSEL button every time you want to flash your program. But the Debug Probe offers much more than that. It allows you to use GDB directly from your computer, so you can debug your program while it is running on the Pico 2.

## What is GDB?

If you have never used GDB, here is a simple explanation: GDB is a command line debugger that lets you pause your program, inspect what is happening inside it, read memory, and step through the code to find problems.

For debugging the Pico 2, you need a version of GDB that supports ARM targets. You can install it with:

```sh
sudo apt install gdb-multiarch
```

## Enable GDB in Embed.toml

Earlier, we used probe-rs through the cargo embed command. The same tool can also start a GDB server, which lets you connect GDB to the Pico 2 through the Debug Probe.

For this, we need to edit the `Embed.toml` file in the root of your project. This file is the configuration file used by the cargo embed command. You should add the following section to enable the GDB server:

```toml
[default.gdb]
# Whether or not a GDB server should be opened after flashing.
enabled = true
```

## Example Project

For this exercise, I have created a simple LED blink program using rp-hal. It does not use Embassy to keep things simple. The `Embed.toml` file is already set up, so you can clone the project and start working right away:

```sh
git clone https://github.com/ImplFerris/pico-debug
cd pico-debug
```

If you run the `cargo embed` command now, the GDB server will start automatically and listen on port 1337 (the default port used by `probe-rs`).

## Connecting GDB to the Remote Server

To connect GDB to the running probe-rs GDB server, open a new terminal and start GDB with the our project binary file:

> Note: There is an issue with probe-rs version 0.30. When I try to connect to the GDB server, the connection closes immediately. I downgraded to version 0.28 as suggested in this [issue discussion](https://github.com/probe-rs/probe-rs/issues/3633). After downgrading, run cargo embed again.

```sh
gdb-multiarch ./target/thumbv8m.main-none-eabihf/debug/pico-debug
```

Then connect to the server on port 1337:

```sh
(gdb) target remote :1337
```

At this point, GDB is connected to the Pico 2 through the Debug Probe, and you can start using breakpoints, stepping, memory inspection, and other debugging commands.

## Resetting to the Start of the Program

When you connect GDB to the running GDB server, the CPU may not be stopped at the start of your program. It might be sitting somewhere deep inside the code.

To ensure you start debugging from a clean state, run:

```sh
(gdb) monitor reset halt
```

This command tells the Debug Probe to reset the Pico 2 and immediately halt the CPU. This puts the program back at the very beginning, right where the processor starts running after a reset.
