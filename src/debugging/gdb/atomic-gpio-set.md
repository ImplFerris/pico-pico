# Atomic GPIO Register

Earlier, we looked only at the GPIO_OUT register. That register holds the full 32-bit output value for all GPIO pins. But in practice, the rp-hal library does not write to GPIO_OUT directly. Instead, it uses the atomic helper registers: GPIO_OUT_SET, GPIO_OUT_CLR, and GPIO_OUT_XOR.

These atomic registers are write-only registers within the SIO block that don't hold values themselves. When you write to them, the bits you set are used to modify the underlying GPIO_OUT register:

- GPIO_OUT_SET changes specified bits to 1. This register is at address `0xd0000018`, as per the datasheet.
- GPIO_OUT_CLR changes specified bits to 0. This register is at address `0xd0000020`, as per the datasheet.
- GPIO_OUT_XOR toggles specified bits

Only the bits that we write as 1 are changed. All other bits stay untouched. This makes it safer and prevents accidental changes to other pins.

For example, if we want to control GPIO25:

- To set GPIO25 high, we write a 1 to bit 25 of GPIO_OUT_SET. So the GPIO_OUT_SET value will be
`0b00000010_00000000_00000000_00000000` (or in hex `0x02000000`).

- To set GPIO25 low, we write a 1 to bit 25 of GPIO_OUT_CLR. So the GPIO_OUT_CLR value will be
`0b00000010_00000000_00000000_00000000` (or in hex `0x02000000`).

These operations modify only bit 25 in GPIO_OUT, leaving all other bits intact.

## Inside rp-hal: Setting a Pin High or Low

If we follow what `set_high()` and `set_low()` do inside `rp-hal`, we can see that they never write to GPIO_OUT directly. Instead, they write to the atomic registers GPIO_OUT_SET and GPIO_OUT_CLR.

The [code inside rp-hal](https://github.com/rp-rs/rp-hal/blob/28fdf0c3b3bfe67d1ceae92050ccbb469bff3a29/rp235x-hal/src/gpio/mod.rs#L672) looks like this:

```rust
 #[inline]
pub(crate) fn _set_low(&mut self) {
    let mask = self.id.mask();
    self.id.sio_out_clr().write(|w| unsafe { w.bits(mask) });
}

#[inline]
pub(crate) fn _set_high(&mut self) {
    let mask = self.id.mask();
    self.id.sio_out_set().write(|w| unsafe { w.bits(mask) });
}
```

When these `write()` functions run, they eventually call `core::ptr::write_volatile()`. `write_volatile` does some pre-checks, and then the compiler's intrinsic `intrinsics::volatile_store` performs the final store to the MMIO address. That volatile store is the moment the actual hardware register changes.

Now let's check how this looks when we step through it in GDB.

## Breakpoint at write_volatile

There are many ways to reach `write_volatile`. One way is to step through `set_low()` or `set_high()` using `stepi` and `nexti` in GDB. But we will take a shorter path. We will set a breakpoint directly on `core::ptr::write_volatile`.

There is one thing to keep in mind. If you set this breakpoint right after reset (for example, right after `monitor reset halt`), GDB will stop many times. This is because `write_volatile` is used in a lot of places during startup. So we will not set it at the beginning.

Instead, follow the steps from the previous chapter. When the program stops at the first breakpoint in your code, like this:

```sh
Continuing.

Thread 1 hit Breakpoint 1, 0x100002f8 in pico_debug::__cortex_m_rt_main () at src/main.rs:63
63              led_pin.set_high().unwrap();
```

> [!TIP]
> You can check your breakpoints with `info break`. You can delete the breakpoint with `delete <number>`.

Now that we're past the startup code, let's set our breakpoint on `write_volatile`:

```sh
(gdb) break core::ptr::write_volatile
```

Then continue execution:

```sh
(gdb) continue
```

You should see output similar to this:

```sh
Thread 1 received signal SIGINT, Interrupt.
rp235x_hal::gpio::eh1::{impl#1}::set_high<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::pull::PullDown> (self=0x2007ffbd)
    at /home/implrust/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/rp235x-hal-0.3.1/src/gpio/mod.rs:1549
1549            fn set_high(&mut self) -> Result<(), Self::Error> {
```

Continue again:
```sh
(gdb) continue
```

Now we've stopped inside the `write_volatile` function:

```sh
Thread 1 hit Breakpoint 3, core::ptr::write_volatile<u32> (dst=0xd0000018, src=33554432)
    at /home/implrust/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ub_checks.rs:76
76                  if ::core::ub_checks::$kind() {
```

Did you notice the function arguments here? The destination dst is `0xd0000018`, which is the address of the GPIO_OUT_SET register. The source value src is `33554432`. If we convert that to hexadecimal, we get `0x02000000`. In binary, that's `0b00000010_00000000_00000000_00000000`. This is the exact bit mask for GPIO25.

Let's disassemble the function to see what's happening at the assembly level:

```sh
(gdb) disas
Dump of assembler code for function _ZN4core3ptr14write_volatile17hc4948e781ca030f6E:
   0x10008084 <+0>:     push    {r7, lr}
   0x10008086 <+2>:     mov     r7, sp
   0x10008088 <+4>:     sub     sp, #24
   0x1000808a <+6>:     str     r2, [sp, #4]
   0x1000808c <+8>:     str     r1, [sp, #8]
   0x1000808e <+10>:    str     r0, [sp, #12]
   0x10008090 <+12>:    str     r0, [sp, #16]
   0x10008092 <+14>:    str     r1, [sp, #20]
=> 0x10008094 <+16>:    b.n     0x10008096 <_ZN4core3ptr14write_volatile17hc4948e781ca030f6E+18>
   0x10008096 <+18>:    ldr     r2, [sp, #4]
   0x10008098 <+20>:    ldr     r0, [sp, #12]
   0x1000809a <+22>:    movs    r1, #4
   0x1000809c <+24>:    bl      0x100080ac <_ZN4core3ptr14write_volatile18precondition_check17h8beabfccc7ba3236E>
   0x100080a0 <+28>:    b.n     0x100080a2 <_ZN4core3ptr14write_volatile17hc4948e781ca030f6E+30>
   0x100080a2 <+30>:    ldr     r0, [sp, #8]
   0x100080a4 <+32>:    ldr     r1, [sp, #12]
   0x100080a6 <+34>:    str     r0, [r1, #0]
   0x100080a8 <+36>:    add     sp, #24
   0x100080aa <+38>:    pop     {r7, pc}
End of assembler dump.
```

The key instruction is at address `0x100080a6`. This is the line that actually writes to the hardware register. At this point, `r1` will contain the GPIO_OUT_SET address and `r0` will contain the value that is going to be written.

Let's take a closer look. We set another breakpoint right on that instruction:

```sh
(gdb) break *0x100080a6
```

Then continue:

```sh
(gdb) continue
```

If you get interrupted, continue again

```sh
Thread 1 received signal SIGINT, Interrupt.
core::ptr::write_volatile<u32> (dst=0xd0000018, src=33554432)
    at /home/implrust/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ub_checks.rs:77
77                      precondition_check($($arg,)*);
```

Continue again:

```sh
(gdb) c
Continuing.

Thread 1 hit Breakpoint 4, 0x100080a6 in core::ptr::write_volatile<u32> (dst=0xd0000018, src=33554432)
    at /home/implrust/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ptr/mod.rs:2201
2201            intrinsics::volatile_store(dst, src);
```

GDB will stop exactly at the store instruction. If you run disas again, you'll see the arrow pointing to that line:

```sh
...
   0x100080a4 <+32>:    ldr     r1, [sp, #12]
=> 0x100080a6 <+34>:    str     r0, [r1, #0]
   0x100080a8 <+36>:    add     sp, #24
```

Before we execute this write instruction, let's check what values are in registers r0 and r1:

```sh
(gdb) i r $r0
r0             0x2000000           33554432

(gdb) i r $r1
r1             0xd0000018          3489660952
```

Let's also examine the current value in the GPIO_OUT register:

```sh
(gdb) x/x 0xd0000010
0xd0000010:     0x00000000
```

Right now it shows all zeros. At this stage, the LED is still off because we haven't executed the store instruction yet.r

Now let's step forward by one instruction:

```sh
(gdb) nexti

#or

(gdb) ni
```

After executing this command, you should see the LED turn on. Now let's examine the GPIO_OUT register again:

```sh
(gdb) x/x 0xd0000010
0xd0000010:     0x02000000
```

The register now shows `0x02000000`, which is exactly the bit mask for GPIO25. This confirms that our write operation successfully set the LED pin high.

## Your Turn: Try It Yourself

Now it's time to practice what you've learned. Let the program continue running until it hits the `set_low` breakpoint. Then continue execution again until you reach the `write_volatile` function.

This time, things will be a bit different. The destination address will be `0xd0000020`, which is the GPIO_OUT_CLR register. As the name suggests, this register is used to clear GPIO pins rather than set them.

Step through the code just like before. When you execute the str instruction, the LED will turn off. If you examine the GPIO_OUT register afterwards, you'll see it contains all zeros again. This confirms that the bit for GPIO25 has been cleared, turning off the LED.
