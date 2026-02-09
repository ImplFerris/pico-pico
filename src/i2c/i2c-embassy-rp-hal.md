# I2C in Embassy RP

Let's see how to initialize and use I2C with Embassy on the Raspberry Pi Pico 2.

## Blocking mode

Embassy provides a simple way to set up I2C in blocking mode:

```rust
let sda = p.PIN_16;
let scl = p.PIN_17;

info!("set up i2c");
let mut i2c = i2c::I2c::new_blocking(p.I2C0, scl, sda, Config::default());
```

We use the `new_blocking` method to create an I2C instance that waits for each operation to finish before continuing. First we choose which I2C peripheral we want to work with, either I2C0 or I2C1. Once we select the peripheral, we must pair it with the correct GPIO pins for SCL and SDA.

For the configuration, the default implementation gives us standard 100 kHz communication and also enables internal pullups.

## Customizing Config

The `Config` struct lets us control how the I2C bus behaves. We can adjust the communication speed and whether the internal pullups on the SDA and SCL lines are enabled.

If we want to increase the bus speed, we can change the frequency field:

```rust
let mut config = Config::default();
config.frequency = 400_000;
```

If our circuit already includes external pullup resistors, we can disable the internal ones:

```rust
let mut config = Config::default();
config.sda_pullup = false;
config.scl_pullup = false;
```

## Sending Data

Many I2C devices require us to send commands or configuration bytes. For example, imagine we are configuring a sensor and need to `write` two bytes to it:

```rust
const SENSOR_ADDR: u8 = 0x68;
let config_data = [0x6B, 0x00];

i2c.write(SENSOR_ADDR, &config_data)?;
```

Here, we're sending two bytes to the device at address `0x68`. The first byte `0x6B` typically tells the device which register we're writing to, and `0x00` is the value we want to write. Different devices use this pattern differently, so you'll need to check your device's datasheet to know what bytes to send.

## Reading from a Register

Most I2C devices store their data in registers. To read a specific register, we use `write_read`. Let's say we want to read the temperature from a sensor:

```rust
const TEMP_REGISTER: u8 = 0x41;
let mut buffer = [0u8; 2];

i2c.write_read(SENSOR_ADDR, &[TEMP_REGISTER], &mut buffer)?;
```

We first tell the device "I want to read from register `0x41`" (the write part), then the device sends us back 2 bytes of temperature data (the read part). The `write_read` method does both operations in a single I2C transaction. After this, our `buffer` will contain the raw temperature bytes that we can then convert to an actual temperature value.

## Reading Continuously

Some devices automatically advance their internal pointer and keep producing data. For these cases we can use a simple `read`:

```rust
let mut buffer = [0u8; 5];
i2c.read(SENSOR_ADDR, &mut buffer)?;
```

This reads bytes starting from the device's current internal position. It is less common than `write_read`, but useful for sensors that stream data continuously.

## Using Async Mode

If we're building a more complex application that needs to handle multiple things at once, we can use async mode. This lets our program do other work while waiting for I2C operations to complete:

```rust
bind_interrupts!(struct Irqs {
    I2C0_IRQ => i2c::InterruptHandler<I2C0>;
});

let mut i2c = I2c::new_async(
    p.I2C0,
    scl,
    sda,
    Irqs,
    I2cConfig::default(),
);

let mut buffer = [0u8; 2];

i2c.write_read(SENSOR_ADDR, &[TEMP_REGISTER], &mut buffer).await?;
```

## Target (Slave) mode

The Pico can also act as an I2C target device (also known as a slave device), where it responds to requests from another controller. However, for most of our projects in this book, we'll be using the Pico as the controller that talks to sensors and other peripherals, so we won't cover target mode here.
