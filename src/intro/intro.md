# Introduction

If you haven't read "The Embedded Rust Book" yet, I highly recommend you to check it out.
[https://docs.rust-embedded.org/book/intro/index.html](https://docs.rust-embedded.org/book/intro/index.html)

## #![no_std]
The `#![no_std]` attribute disables the use of the standard library (std). This is necessary most of the times for embedded systems development, where the environment typically lacks many of the resources (like an operating system, file system, or heap allocation) that the standard library assumes are available.

**Related Resources:**
- [Rust official doc](https://doc.rust-lang.org/reference/names/preludes.html#the-no_std-attribute)
- [The Embedded Rust Book](https://docs.rust-embedded.org/book/intro/no-std.html)
- [Section from 'Writing an OS in Rust ' Book](https://os.phil-opp.com/freestanding-rust-binary/#the-no-std-attribute)

## #![no_main]
The `#![no_main]` attribute is to indicate that the program won't use the standard entry point (fn main). Instead, it provides a custom entry point, usually required when working with embedded systems where the runtime environment is minimal or non-existent.

**Related Resources:**
- [Rust official doc](https://doc.rust-lang.org/reference/crates-and-source-files.html?highlight=no_main#the-no_main-attribute)
- [Section from 'Writing an OS in Rust ' Book](https://os.phil-opp.com/freestanding-rust-binary/#overwriting-the-entry-point)
