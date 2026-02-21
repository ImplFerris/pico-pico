# Pico Pico - Intro

In this book, we use the Raspberry Pi Pico 2 and program it in Rust to explore various exciting projects. You'll work on exercises like dimming an LED, controlling a servo motor, measuring distance with a sonic sensor, displaying the Ferris (🦀) image on an OLED display, using RFID reader, playing songs on a buzzer, turning on an LED when the room light is off, measuring temperature, and much more.

## Meet the hardware - Pico 2

We will be using the Raspberry Pi Pico 2 (RP2350 chip). It features the new RP2350 chip with dual-core flexibility;offering Arm Cortex-M33 cores and optional RISC-V Hazard3 cores. You can operate it as the standard ARM core or switch to the RISC-V architecture. You find more details from the [official website](https://www.raspberrypi.com/products/raspberry-pi-pico-2/).

<img style="display: block; margin: auto;" alt="pico2" src="./src/images/pico2.png"/>

> There is an older Raspberry Pi Pico that uses the RP2040 chip. In this book, we will be using the newer **Pico 2** with the **RP2350** chip. When buying hardware, make sure to get the correct one!
>
> If you are looking for the RP2040 version of this book, you can find it here: [impl Rust for Pico](http://rp2040.implrust.com/)


## How to read

You can access the MD book here: https://pico.implrust.com/

or you can run locally

```sh
mdbook serve --open
```

## Support this project

You can support this book by starring this project on [GitHub](https://github.com/ImplFerris/pico-pico) or sharing this book with others 😊

## License

The Pico Pico book(this project) is distributed under the following licenses:

* The code samples and free-standing Cargo projects contained within this book are licensed under the terms of both the [MIT License] and the [Apache License v2.0].
* The written prose contained within this book is licensed under the terms of the Creative Commons [CC-BY-SA v4.0] license.

[MIT License]: ./LICENSE-MIT
[Apache License v2.0]: ./LICENSE-APACHE
[CC-BY-SA v4.0]: ./LICENSE-CC-BY-SA
[MIT License Hosted]: https://opensource.org/licenses/MIT
[Apache License v2.0 Hosted]: http://www.apache.org/licenses/LICENSE-2.0
[CC-BY-SA v4.0 Hosted]: https://creativecommons.org/licenses/by-sa/4.0/legalcode


### Disclaimer:

The experiments and projects shared in this book have worked for me, but results may vary. I'm not responsible for any issues or damage that may occur while you're experimenting. Please proceed with caution and take necessary safety precautions.
