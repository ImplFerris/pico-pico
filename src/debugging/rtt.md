# Real-Time Transfer (RTT)

When developing embedded systems, you need a way to see what's happening inside your program. On a normal computer, you would use println! to print messages to the terminal. But on a microcontroller, there's no screen or terminal attached. Real-Time Transfer (RTT) solves this problem by letting you print debug messages and logs from your microcontroller to your computer.

## What is RTT?

RTT is a communication method that lets your microcontroller send messages to your computer through the debug probe you're already using to flash your programs. 

When you connect the Raspberry Pi Debug Probe to Pico, you're creating a connection that can do two things:

- Flash new programs onto the chip
- Read and write the chip's memory

RTT uses this memory access capability. It creates special memory buffers on your microcontroller, and the debug probe reads these buffers to display messages on your computer. This happens in the background while your program runs normally.

## Using Defmt for Logging

[Defmt](https://github.com/knurling-rs/defmt) (short for "deferred formatting") is a logging framework designed specifically for resource-constrained devices like microcontrollers.  In your Rust embedded projects, you'll use defmt to print messages and debug your programs.

Defmt achieves high performance using deferred formatting and string compression. Deferred formatting means that formatting is not done on the machine that's logging data but on a second machine. 

Your Pico sends small codes instead of full text messages. Your computer receives these codes and turns them into normal text. This keeps your firmware small and avoids slow string formatting on the microcontroller.

You can add the defmt crate in your project:

```toml
defmt = "1.0.1"
```

Then use it like this:

```rust
use defmt::{info, warn, error};

...
info!("Starting program");
warn!("You shall not pass!");
error!("Something went wrong!");
```

### Defmt RTT

By itself, defmt doesn't know how to send messages from your Pico to your computer. It needs a transport layer. That's where defmt-rtt comes in.

The defmt-rtt crate connects defmt to RTT, so your log messages get transmitted through the debug probe to your computer.

You can add the defmt-rtt crate in your project:

```toml
defmt-rtt = "1.0"
```

> Note: To see RTT and defmt logs, you need to run your program using probe-rs tools like the `cargo embed` command. These tools automatically open an RTT session and show the logs in your terminal

Then include it in your code:

```rust
use defmt_rtt as _;
```

The line sets up the connection between defmt and RTT. You don't call any functions from it directly, but it needs to be imported to make it work.

### Panic Messages with Panic-Probe

When your program crashes (panics), you want to see what went wrong. The panic-probe crate makes panic messages appear through defmt and RTT.

You can add the panic-probe crate in your project:

```toml
# The print-defmt feature - tells panic-probe to use defmt for output.
panic-probe = { version = "1.0", features = ["print-defmt"] }
```

Then include it in your code:

```rust
use panic_probe as _;
```

You can manually trigger a panic to see how panic messages work. Try adding this to your code:

```rust
panic!("something went wrong");
```
