# Abstraction Layers

When working with embedded Rust, you will often come across terms like PAC, HAL, and BSP. These are the different layers that help you interact with the hardware. Each layer offers a different balance between flexibility and ease of use.

Let's start from the highest level of abstraction down to the lowest.
 
<a href ="./images/abstraction-layers.png"><img alt="abstraction layers" style="display: block; margin: auto;" src="./images/abstraction-layers.png"/></a>
 

## Board Support Package (BSP)

A BSP, also referred as Board Support Crate in Rust, tailored to specific development boards.  It combines the HAL with board-specific configurations, providing ready to use interfaces for onboard components like LEDs, buttons, and sensors. This allows developers to focus on application logic instead of dealing with low-level hardware details. Since there is no popular BSP specifically for the Raspberry Pi Pico 2, we will not be using this approach in this book.

---

## Hardware Abstraction Layer (HAL)

The HAL sits just below the BSP level. If you work with boards like the Raspberry Pi Pico or ESP32 based boards, you'll mostly use the HAL level. HALs are typically written for the specific chip (like the RP2350 or ESP32) rather than for individual boards, which is why the same HAL can be used across different boards that share the same microcontroller. For Raspberry Pi's family of microcontrollers, there's the [rp-hal](https://github.com/rp-rs/rp-hal) crate that provides this hardware abstraction.

The HAL builds on top of the PAC and provides simpler, higher-level interfaces to the microcontroller's peripherals. Instead of handling low-level registers directly, HALs offer methods and traits that make tasks like setting timers, setting up serial communication, or controlling GPIO pins easier.

HALs for the microcontrollers usually implement the `embedded-hal` traits, which are standard, platform-independent interfaces for peripherals like GPIO, SPI, I2C, and UART. This makes it easier to write drivers and libraries that work across different hardware as long as they use a compatible HAL.

### Embassy for RP

Embassy sits at the same level as HAL but provides an additional runtime environment with async capabilities. Embassy (specifically embassy-rp for Raspberry Pi Pico) is built on top of the HAL layer and provides an async executor, timers, and additional abstractions that make it easier to write concurrent embedded applications.

Embassy provides a separate crate called `embassy-rp` specifically for Raspberry Pi microcontrollers (RP2040 and RP235x). This crate builds directly on top of the rp-pac (Raspberry Pi Peripheral Access Crate).

Throughout this book, we will use both rp-hal and embassy-rp for different exercises.
 
---
> NOTE:
> 
> The layers below the HAL are rarely used directly. In most cases, the PAC is accessed through the HAL, not on its own. Unless you are working with a chip that does not have a HAL available, there is usually no need to interact with the lower layers directly. In this book, we will focus on the HAL layer.


## Peripheral Access Crate (PAC)

PACs are the lowest level of abstraction. They are auto-generated crates that provide type-safe access to a microcontroller's peripherals. These crates are typically generated from the manufacturer's SVD (System View Description) file using tools like `svd2rust`. PACs give you a structured and safe way to interact directly with hardware registers.

## Raw MMIO

Raw MMIO (memory-mapped IO) means directly working with hardware registers by reading and writing to specific memory addresses.  This approach mirrors traditional C-style register manipulation and requires the use of `unsafe` blocks in Rust due to the potential risks involved.  We will not touch this area; I haven't seen anyone using this approach.
