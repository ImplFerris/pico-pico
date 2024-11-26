# USB Serial Communication

In this section, we'll explore how to establish communication between our device (Pico) and a computer(Linux). We'll demonstrate how to send a simple string from the device(Pico) to the computer, as well as how to send input from the computer to the device.

## CDC ACM

The Communication Device Class (CDC) is a standard USB device class [defined](https://www.usb.org/document-library/class-definitions-communication-devices-12) by the USB Implementers Forum (USB-IF). The Abstract Control Model (ACM) in CDC allows a device to act like a traditional serial port (like old COM ports). It's commonly used for applications that previously relied on serial COM or UART communication.

## Tools for Linux
When you flash the code in this exercise, the device will appear as /dev/ttyACM0 in your computer. To interact with the USB serial port on Linux, you can use tools like minicom, tio (or cat ) to read and send data to and from the device

- [minicom](https://help.ubuntu.com/community/Minicom): Minicom is a text-based serial port communications program. It is used to talk to external RS-232 devices such as mobile phones, routers, and serial console ports.
- [tio](https://github.com/tio/tio): tio is a serial device tool which features a straightforward command-line and configuration file interface to easily connect to serial TTY devices for basic I/O operations.

## Rust Crates
We will be using the example taken from the RP-HAL repository. It use two crates: [usb-device](https://crates.io/crates/usb-device), an USB stack for embedded devices in Rust, and [usbd-serial](https://crates.io/crates/usbd-serial), which implements the USB CDC-ACM serial port class. The SerialPort class in usbd-serial implements a stream-like buffered serial port and can be used in a similar way to UART.

## References
- [CDC: Communication Device Class (ACM)](https://www.keil.com/pack/doc/mw/usb/html/group__usbd__cdc_functions__acm.html)
- [USB Device CDC ACM Class](https://docs.silabs.com/protocol-usb/1.2.0/protocol-usb-cdc/)
- [What is the difference between /dev/ttyUSB and /dev/ttyACM?](https://rfc1149.net/blog/2013/03/05/what-is-the-difference-between-devttyusbx-and-devttyacmx/)
- [Defined Class Codes](https://www.usb.org/defined-class-codes)


