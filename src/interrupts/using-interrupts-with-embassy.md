# Using Interrupts with Embassy

In the previous chapter, we looked at what interrupts are and how the NVIC fits into the picture. Now lets see how interrupts are actually used in Embassy.

In Embassy, you normally do not write interrupt handlers yourself. Async drivers use interrupts internally to wait for hardware events and to wake tasks when those events happen. Your code just awaits an operation and continues when it is ready.

For some peripherals, Embassy needs a small amount of setup so it knows which hardware interrupt belongs to which driver. This is where `bind_interrupts!` comes in.

## Why `bind_interrupts!` Is Needed

Async peripherals like I2C, SPI do not finish their work in one step. While an operation is in progress, the task goes to sleep and the hardware generates interrupts as things move forward.

Embassy already provides the interrupt handlers for these peripherals. What it needs from you is the connection between the hardware interrupt and the handler it should use. The `bind_interrupts!` macro is how you make that connection.

You are not writing an interrupt handler here. You are just wiring things up so the async driver can work.

## Binding an Interrupt for I2C

Here is a simple example for I2C:

```rust
use embassy_rp::{bind_interrupts, i2c};
use embassy_rp::peripherals::I2C0;

bind_interrupts!(struct Irqs {
    I2C0_IRQ => i2c::InterruptHandler<I2C0>;
});
```

This tells Embassy that the I2C0_IRQ interrupt should be handled by the I2C driver for I2C0. Once this is in place, async I2C operations can sleep and wake correctly.


## Using Async I2C

After the interrupt is bound, using async I2C looks normal:

```rust
use embassy_rp::i2c::{I2c, Config as I2cConfig};
use embassy_rp::peripherals::I2C0;

let sda = p.PIN_16;
let scl = p.PIN_17;

let mut i2c = I2c::new_async(
    p.I2C0,
    scl,
    sda,
    Irqs,
    I2cConfig::default(),
);
```

When you later call an async operation like:

```rust
i2c.write(0x3C, &[0x00]).await;
```

the task goes to sleep, hardware interrupts fire as the transfer progresses, and Embassy wakes the task when the operation finishes. You never see the interrupt directly, but it is doing the work in the background.
