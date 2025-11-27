# Breakpoints

Now that we've traced the execution path from reset to our main function, let's set breakpoints in the LED loop and observe how the GPIO registers change when we toggle the LED.


## Understanding the LED Loop

Let me show you the disassembled code from the __cortex_m_rt_main function again. We need to look for the `bl` instructions. The `bl` stands for "branch and link" - these are instructions that call other functions. Specifically, we're looking for the calls to set_high and set_low functions.

```rust
...
// This is the set_high() call
0x100002f8 <+404>:   bl      0x10000c48 <rp235x_hal::gpio::eh1::{impl#1}::set_high<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::pull::PullDown>>
...
// This is the delay_ms() call
0x10000306 <+418>:   bl      0x100006b8 <rp235x_hal::timer::{impl#7}::delay_ms<rp235x_hal::timer::CopyableTimer0>>
...
// This is the set_low() call
0x1000030c <+424>:   bl      0x10000c38 <rp235x_hal::gpio::eh1::{impl#1}::set_low<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::pull::PullDown>>
...
// This is the delay_ms() call
0x10000314 <+432>:   bl      0x100006b8 <rp235x_hal::timer::{impl#7}::delay_ms<rp235x_hal::timer::CopyableTimer0>>
...
```

Look at those addresses on the left - 0x100002f8 and 0x1000030c. These are memory addresses where the LED control happens. The first address is where set_high gets called, and the second is where set_low gets called. We're going to put breakpoints at these addresses so our program pauses right before running these instructions.

## Setting Breakpoints in the Loop

Let's set up the first breakpoint. Type this in GDB:

```sh
(gdb) break *0x100002f8
```

You'll see:  `Breakpoint 1 at 0x100002f8: file src/main.rs, line 63.`

This means GDB created a breakpoint at that address, and it corresponds to line 63 in our main.rs file.

```sh
(gdb) break *0x1000030c
```

You'll see: `Breakpoint 2 at 0x1000030c: file src/main.rs, line 65.`


Now let's reset everything to start fresh:

```sh
monitor reset halt
```

This resets the microcontroller and stops it at the beginning, so we have a clean starting point.

## GPIO Register Overview

Before we continue, I need to explain what we're going to look at. When you call set_high or set_low in your Rust code, what actually happens is that specific memory locations get changed. These memory locations are called registers, and they directly control the hardware.

On the RP2350 chip, there's a register called GPIO_OUT that controls all the GPIO pins. You can find this in the RP2350 datasheet (chapter 3.1.11, page 55) under the SIO (Single-cycle IO) section.

Here's where this register lives in memory:

- The SIO peripheral starts at base address 0xd0000000
- The GPIO_OUT register is at offset 0x010 from that base
- So the full address is: 0xd0000000 + 0x010 = 0xd0000010

Think of GPIO_OUT as a 32-bit number where each bit controls one GPIO pin. Bit 0 controls GPIO0, bit 1 controls GPIO1, and so on. Bit 25 controls GPIO25 - that's where the onboard LED is connected. When bit 25 is 0, the LED is off. When bit 25 is 1, the LED is on.

## Running to the First Breakpoint

Let's run the program until it hits our first breakpoint:

```sh
(gdb) continue
```

When the breakpoint is hit, GDB will show something like:

```sh
Continuing.

Thread 1 hit Breakpoint 1, 0x100002f8 in pico_debug::__cortex_m_rt_main () at src/main.rs:63
63              led_pin.set_high().unwrap();
```

The program stopped right before calling set_high. This is the perfect moment to check what the register looks like before we turn the LED on.


## Checking GPIO Registers Before set_high

Let's look at what's currently in the GPIO_OUT register:

```sh
(gdb) x/x 0xd0000010
```

The x/x command means "examine this memory address and show me the value in hexadecimal format."

You'll probably get an error message "Cannot access memory at address 0xd0000010". This happens because GDB doesn't automatically know about peripheral registers. We need to tell GDB that it's allowed to read from this memory region.


### Making SIO Peripheral Accessible in GDB

To fix this, we need to tell GDB about the peripheral memory region. According to the RP2350 datasheet, the SIO region actually extends from 0xd0000000 to 0xdfffffff. However, we don't need to map the entire SIO region - we only need enough to cover the registers we want to access.

So we can type:

```sh
(gdb) mem 0xD0000000 0xD0001000 rw nocache
```

Here, we're mapping about 4KB of the SIO region (from 0xD0000000 to 0xD0001000), which is more than enough to cover GPIO_OUT and the other SIO registers we'll be looking at during debugging.

If you want to map even less and be more precise, you can use:

```sh
(gdb) mem 0xD0000000 0xD0000100 rw nocache
``` 

This gives us just 256 bytes, which covers all the basic SIO registers we need, including GPIO_OUT at 0xD0000010. The key point is that we map enough memory to include the registers we want to read, without needing to map the entire SIO region.

Now try reading GPIO_OUT again:
```sh
(gdb) x/x 0xd0000010
0xd0000010:     0x00000000
```
We get the value 0x00000000. This means all 32 bits are zero, so all GPIO pins are currently off. Our LED is off.

## Continue to the Second Breakpoint

Now let's continue running and see what happens after set_high executes:

```sh
(gdb) continue
Continuing.

Thread 1 received signal SIGINT, Interrupt.
rp235x_hal::gpio::eh1::{impl#1}::set_high<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::pull::PullDown> (self=0x2007ffbd)
    at /home/implrust/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/rp235x-hal-0.3.0/src/gpio/mod.rs:1549
1549            fn set_high(&mut self) -> Result<(), Self::Error> {
```

We got interrupted inside the set_high function. Let's continue again:

```sh
(gdb) continue
Continuing.

Thread 1 hit Breakpoint 2, pico_debug::__cortex_m_rt_main () at src/main.rs:65
65              led_pin.set_low().unwrap();

```

Now the program has run through set_high and the delay, and stopped at our second breakpoint on line 65, right before calling set_low.  Let's check GPIO_OUT again:

```sh
(gdb) x/x 0xd0000010
0xd0000010:     0x02000000
```

The value changed from `0x00000000` to `0x02000000`.  You should also see the LED turned on by this time.

Let me explain what `0x02000000` means. In binary, this is `00000010 00000000 00000000 00000000`. If you count from the right starting at 0, bit 25 is now set to 1. That's exactly what set_high did - it turned on bit 25 of the GPIO_OUT register, which turned on GPIO25, which lit up the LED.


## Continue to See set_low in Action

Now let's continue one more time to see what happens when set_low executes. But first, let's note that the LED is currently on and GPIO_OUT shows `0x02000000` with bit 25 set to 1.

Let's continue:
```sh
(gdb) continue
Continuing.

Thread 1 received signal SIGINT, Interrupt.
rp235x_hal::gpio::eh1::{impl#1}::set_low<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::pull::PullDown> (self=0x2007ffbd)
    at /home/implrust/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/rp235x-hal-0.3.0/src/gpio/mod.rs:1544
1544            fn set_low(&mut self) -> Result<(), Self::Error> {
```

We got interrupted inside the set_low function. Let's continue again:

```sh
(gdb) continue
Continuing.

Thread 1 hit Breakpoint 1, 0x100002f8 in pico_debug::__cortex_m_rt_main () at src/main.rs:63
63              led_pin.set_high().unwrap();
```

The program ran through set_low and the delay, and looped back to our first breakpoint on line 63. Let's check GPIO_OUT again:

```sh
(gdb) x/x 0xd0000010
0xd0000010:     0x00000000
```

The value is back to `0x00000000`. Bit 25 is now 0, which means GPIO25 is off and the LED is off. You should see the LED turned off on your board.


## What We Learned

From what we observed:

- When we call `led_pin.set_high()`, bit 25 of GPIO_OUT changes from 0 to 1 (`0x00000000` → `0x02000000`)
- When we call `led_pin.set_low()`, bit 25 changes from 1 to 0 (`0x02000000` → `0x00000000`)
