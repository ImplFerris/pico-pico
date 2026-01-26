# Finding the Reset Handler and Tracing the Call to main

When the Pico 2 resets, the CPU starts executing from the Reset Handler. To understand how our program starts, we will locate the Reset Handler, disassemble it, and follow the call chain until we reach our actual Rust main.

When the Pico 2 starts up, the CPU does not jump straight into our Rust main function.  Instead, it follows a small chain of functions provided by the Cortex-M runtime.

In this section, we will:

1. Find where the chip starts executing after reset

2. See which function that Reset Handler calls

3. Follow the chain until we reach our real Rust main

## Read the Reset Vector Entry

The Cortex-M processor starts execution by reading a table at the beginning of flash memory called the vector table.

The first two entries are:

- **Word 0** (offset 0x00): Initial stack pointer value
- **Word 1** (offset 0x04): Reset handler address

On Pico 2, flash starts at address 0x10000000 so:

- The initial stack pointer value is stored at 0x10000000
- Reset handler address is at 0x10000004

### What is the Reset Handler?

The reset handler is the first function that runs when the processor powers on or resets. It performs initialization and eventually calls our `main` function.

Read it in GDB:

```sh
(gdb) x/wx 0x10000004
```

Example output - actual addresses might differ at your artifacts, use those to follow this chapters:

```sh
0x10000004 <__RESET_VECTOR>:    0x1000010d
```

This value is the address the CPU jumps to after reset. The last bit (the "Thumb bit") is always 1, so the actual address is `0x1000010c`.
But you can use either one of them (`0x1000010d` or `0x1000010c`), GDB can handle it.

Alternatively, you can also use the `readelf` program to find the entrypoint address:

```sh
arm-none-eabi-readelf -h ./target/thumbv8m.main-none-eabihf/debug/pico-debug
```

## Disassemble the Reset Handler

Now Let's ask GDB to show the instructions at that address:

```sh
(gdb) disas 0x1000010d

# or

(gdb) disas 0x1000010c
```

You will see assembly instructions for the reset handler. Look for a `bl` ([Branch with Link](https://developer.arm.com/documentation/100076/0200/a32-t32-instruction-set-reference/a32-and-t32-instructions/bl)) instruction that calls another function:

```sh
...
0x10000140 <+52>:    isb     sy
0x10000144 <+56>:    bl      0x1000031c <main>
0x10000148 <+60>:    udf     #0
```

The Reset Handler calls a function located at `0x1000031c`, which GDB shows as main. But this is not our Rust main yet.

## What is this "main"?

The main at `0x1000031c` is not our program's main function. It is a small wrapper created by the `cortex-m-rt` crate. This wrapper is often called the trampoline because it jumps to the real entry point later.

Its demangled name is usually:

```rust
// NOTE: here, pico_debug prefix is our project's name
pico_debug::__cortex_m_rt_main_trampoline
```

Let's disassemble it.

### Disassemble that trampoline

```sh
(gdb) disas 0x1000031c
```

Output:

```sh
Dump of assembler code for function main:
   0x1000031c <+0>:     push    {r7, lr}
   0x1000031e <+2>:     mov     r7, sp
   0x10000320 <+4>:     bl      0x10000164 <_ZN10pico_debug18__cortex_m_rt_main17he0b4d19700c84ad2E>
End of assembler dump.
```

This is very small.  All it does is call the real Rust entrypoint, which is named:

```rust
pico_debug::__cortex_m_rt_main
```

## Enable Demangled Names

Rust function names are mangled by default and look unreadable.

Enable demangling:

```sh
set print asm-demangle on
```

Now try:

```sh
(gdb) disas 0x1000031c
#or
(gdb) disas pico_debug::__cortex_m_rt_main_trampoline
```

You should now see readable Rust names.

```sh
Dump of assembler code for function pico_debug::__cortex_m_rt_main_trampoline:
   0x1000031c <+0>:     push    {r7, lr}
   0x1000031e <+2>:     mov     r7, sp
   0x10000320 <+4>:     bl      0x10000164 <pico_debug::__cortex_m_rt_main>
End of assembler dump.
```

## Disassemble the Actual Rust main

Now let's inspect our main function:

```sh
disas pico_debug::__cortex_m_rt_main
```

You will see the program's logic, starting with the initial setup code followed by the loop that toggles the LED Pin.

```rust
...

0x100002dc <+376>:   bl      0x100079a4 <rp235x_hal::timer::Timer<rp235x_hal::timer::CopyableTimer0>::new_timer0>
0x100002e0 <+380>:   bl      0x10000b30 <rp235x_hal::gpio::Pin<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::func::FunctionNull, rp235x_hal::gpio::pull::PullDown>::into_push_pull_output<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::func::FunctionNull, rp235x_hal::gpio::pull::PullDown>>
...
0x100002f8 <+404>:   bl      0x10000c48 <rp235x_hal::gpio::eh1::{impl#1}::set_high<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::pull::PullDown>>
...
0x10000306 <+418>:   bl      0x100006b8 <rp235x_hal::timer::{impl#7}::delay_ms<rp235x_hal::timer::CopyableTimer0>>
...
0x1000030c <+424>:   bl      0x10000c38 <rp235x_hal::gpio::eh1::{impl#1}::set_low<rp235x_hal::gpio::pin::bank0::Gpio25, rp235x_hal::gpio::pull::PullDown>>
...
```
