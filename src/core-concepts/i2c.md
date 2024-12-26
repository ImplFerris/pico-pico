# I2C

I²C, also known as I2C or IIC (Inter-Integrated Circuit), is a communication protocol widely used to link microcontrollers with devices like sensors, displays, and other integrated circuits. This protocol is also called the Two-Wire Interface (TWI).

- Two Wires: Only 2 lines, SDA (Serial Data) and SCL (Serial Clock), are used to transfer data. 
    - The SDA line is used by both the master and slave to send and receive data
    - The SCL line carries the clock signal

- Multi-Device Support: I2C allows multiple slave devices to be connected to a single master, and it also supports multiple masters.
 
- The master is responsible for generating the clock and controlling the transfer of data, while the slave responds by either transmitting or receiving data from the master.

## RP2350's I2C
The RP2350 has 2 identical I2C controllers, each connected to specific GPIO pins on the Raspberry Pi Pico.

| I2C Controller | GPIO Pins                       |
|----------------|---------------------------------|
| I2C0 – SDA     | GP0, GP4, GP8, GP12, GP16, GP20 |
| I2C0 – SCL     | GP1, GP5, GP9, GP13, GP17, GP21 |
| I2C1 – SDA     | GP2, GP6, GP10, GP14, GP18, GP26|
| I2C1 – SCL     | GP3, GP7, GP11, GP15, GP19, GP27| 

<br/>
<a href="../images/pico2-board.png"><img style="display: block; margin: auto;" alt="pico2" src="../images/pico2-board.png"/></a>

## Resources
- [Basics of the I2C Communication Protocol](https://www.circuitbasics.com/basics-of-the-i2c-communication-protocol/)
- [A Basic Guide to I2C](https://www.ti.com/lit/an/sbaa565/sbaa565.pdf)
- [I2C in a Nutshell](https://interrupt.memfault.com/blog/i2c-in-a-nutshell)
- [I2C](https://learn.sparkfun.com/tutorials/i2c/all)
