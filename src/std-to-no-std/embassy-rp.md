# Embassy for Raspberry Pi Pico

We already introduced the concept of HAL in the introduction chapter. For the Pico, we will use the Embassy RP HAL. The Embassy RP HAL targets the Raspberry Pi RP2040, as well as RP235x microcontrollers.

The HAL supports blocking and async peripheral APIs. Using async APIs is better because the HAL automatically handles waiting for peripherals to complete operations in low power mode and manages interrupts, so you can focus on the primary functionality.

Let's add the embassy-rp crate to our project.

```toml
embassy-rp = { version = "0.8.0", features = [
  "rp235xa",
] }
```

We've enabled the `rp235xa` feature because our chip is the RP2350. If we were using the older Pico, we would instead enable the `rp2040` feature.

## Initialize the embassy-rp HAL

Let's initialize the HAL. We can pass custom configuration to the initialization function if needed. The config currently allows us to modify clock settings, but we'll stick with the defaults for now:

```rust
let peripherals = embassy_rp::init(Default::default());
```

This gives us the peripheral singletons we need. Remember, we should only call this once at startup; calling it again will cause a panic.

## Timer

We are going to replicate the quick start example by blinking the onboard LED. To create a blinking effect, we need a timer to add delays between turning the LED on and off. Without delays, the blinking would be too fast to see.

To handle timing, we'll use the `embassy-time` crate, which provides essential timing functions:

```rust
embassy-time = { version = "0.5.0" }
```

We also need to enable the time-driver feature in the `embassy-rp` crate. This configures the TIMER peripheral as a global time driver for embassy-time, running at a tick rate of 1 MHz:

```toml
embassy-rp = { version = "0.8.0", features = [
  "rp235xa",
  "time-driver",
  "critical-section-impl",
] }
```

We've almost added all the essential crates. Now let's write the code for the blinking effect.
