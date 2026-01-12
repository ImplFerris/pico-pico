# USB Serial Communication

In this section, we are going to set up communication between our device (Pico) and a computer running Linux. We will send a simple string from the Pico to the computer, and we will also send input from the computer back to the Pico.

This is especially useful when you do not have a debug probe and want to print sensor readings or status messages to the system console to quickly see what is going on. I have personally used this approach many times before I actually bought a debug probe.

> [!Tip]
> If you are using a debug probe, you can skip this chapter for now. You can always come back to it later when you want to exchange data with a computer using USB, such as sending messages, reading input, or building simple command based interfaces.

## CDC ACM

When you plug a USB device into a computer, the computer needs to know what kind of device it is and how to talk to it. USB solves this by defining standard device types, called classes.

[CDC](https://www.usb.org/document-library/class-definitions-communication-devices-12) stands for Communication Device Class. It is a USB class meant for devices that communicate by sending and receiving data, similar to serial communication. One very common CDC type is called ACM, which stands for Abstract Control Model.

CDC ACM makes a USB device look like a simple serial port to the computer.

So even though the data is traveling over USB, the operating system treats it like a normal serial connection. On Linux, this is why the device shows up as something like /dev/ttyACM0.

If you have ever used UART with a USB to serial adapter, this feels almost exactly the same from the software side.

## Tools for Linux

When you flash the code in this exercise, the device will appear as /dev/ttyACM0 on your computer. To interact with the USB serial port on Linux, you can use tools like minicom, tio (or cat) to read and send data to and from the device

- [minicom](https://help.ubuntu.com/community/Minicom): Minicom is a text-based serial port communications program. It is used to talk to external RS-232 devices such as mobile phones, routers, and serial console ports.
- [tio](https://github.com/tio/tio): tio is a serial device tool which features a straightforward command-line and configuration file interface to easily connect to serial TTY devices for basic I/O operations.

## Rust Crates

We will be using the example taken from the RP-HAL repository. It uses two crates: [usb-device](https://crates.io/crates/usb-device), an USB stack for embedded devices in Rust, and [usbd-serial](https://crates.io/crates/usbd-serial), which implements the USB CDC-ACM serial port class. The SerialPort class in usbd-serial implements a stream-like buffered serial port and can be used in a similar way to UART.

## References

- [CDC: Communication Device Class (ACM)](https://www.keil.com/pack/doc/mw/usb/html/group__usbd__cdc_functions__acm.html)
- [USB Device CDC ACM Class](https://docs.silabs.com/protocol-usb/1.2.0/protocol-usb-cdc/)
- [What is the difference between /dev/ttyUSB and /dev/ttyACM?](https://rfc1149.net/blog/2013/03/05/what-is-the-difference-between-devttyusbx-and-devttyacmx/)
- [Defined Class Codes](https://www.usb.org/defined-class-codes)
