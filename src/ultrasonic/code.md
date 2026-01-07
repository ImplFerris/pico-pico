# Rust Tutorial: Using the HC-SR04 Sensor with the Pico 2

We will start by creating a new project using the Embassy framework. After that, we wll build the same project again using rp-hal. As usual, generate the project from the template with cargo-generate:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```

When prompted, give your project a name like "bat-beacon" and choose "embassy" as the HAL. Enable defmt logging, if you have a debug probe so you can view logs also.

## Additional Imports

In addition to the usual boilerplate imports, you'll need to add these specific imports to your project.  Your code editor should provide auto-import suggestions for most of these, with the exception of the SetDutyCycle trait which you'll need to add manually.

```rust
// For GPIO
use embassy_rp::gpio::{Input, Level, Output, Pull};

// For PWM
use embassy_rp::pwm::{Pwm, SetDutyCycle};

// For time calculation
use embassy_time::Instant;
```

We need GPIO types to control our trigger and echo pins, PWM to control the LED brightness, and timing utilities to measure the ultrasonic pulse duration.


## Mapping GPIO Pins

By now, you should be familiar with PWM from the Dimming LED section. We will create a similar dimming effect here. But there's a key difference.  In the Dimming LED chapter, we made the LED fade in and out repeatedly using conditions. Here, we will increase the LED brightness only when an object gets closer to the sensor.

```rust
// For Onboard LED
// let mut led = Pwm::new_output_b(p.PWM_SLICE4, p.PIN_25, Default::default());

// For external LED connected on GPIO 3
let mut led = Pwm::new_output_b(p.PWM_SLICE1, p.PIN_3, Default::default());
```
You can use either the onboard LED or an external LED. I prefer using the external LED. You can see the gradual brightness changes much better.

Next, let's initialize the LED to be off and get its maximum duty cycle value:

```rust
led.set_duty_cycle(0)
    .expect("duty cycle is within valid range");

let max_duty = led.max_duty_cycle();
// defmt::info!("Max duty cycle {}", max_duty);
```

The duty cycle determines LED brightness; 0 is completely off, and max_duty is fully on.


## Configuring Trigger and Echo Pins

As you know, we have to send a signal to the trigger pin from the Pico, so we'll configure GPIO pin 17 (connected to the trigger pin) as an Output with an initial Low state. The sensor indicates distance through pulses on the echo pin, meaning it sends signals to the Pico (input to the Pico). So we'll configure GPIO pin 16 (connected to the echo pin) as an Input.

```rust
let mut trigger = Output::new(p.PIN_17, Level::Low);
let echo = Input::new(p.PIN_16, Pull::Down);
```

## Converting Distance to LED Brightness

We need a function that converts distance measurements into appropriate duty cycle values. The closer an object is, the higher the duty cycle (brighter the LED):

```rust
const MAX_DISTANCE_CM: f64 = 30.0;

fn calculate_duty_cycle(distance: f64, max_duty: u16) -> u16 {
    if distance < MAX_DISTANCE_CM && distance >= 2.0 {
        let normalized = (MAX_DISTANCE_CM - distance) / MAX_DISTANCE_CM;
        // defmt::info!("duty cycle :{}", (normalized * max_duty as f64) as u16);
        (normalized * max_duty as f64) as u16
    } else {
        0
    }
}
```

This function takes the measured distance and the maximum duty cycle value. If the distance is between 2cm (the sensor's minimum range) and 30cm, we normalize it to a 0-1 range and multiply by the maximum duty cycle. Objects closer than 2cm or farther than 30cm result in the LED turning off (duty cycle of 0).

## Measuring Distance with the Sensor

We'll measure distance by sending an ultrasonic pulse and timing how long it takes to return:

```rust
const ECHO_TIMEOUT: Duration = Duration::from_millis(100);

async fn measure_distance(trigger: &mut Output<'_>, echo: &Input<'_>) -> Option<f64> {
    // Send trigger pulse
    trigger.set_low();
    Timer::after_micros(2).await;
    trigger.set_high();
    Timer::after_micros(10).await;
    trigger.set_low();

    // Wait for echo HIGH (sensor responding)
    let timeout = Instant::now();
    while echo.is_low() {
        if timeout.elapsed() > ECHO_TIMEOUT {
            defmt::warn!("Timeout waiting for HIGH");
            return None; // Return early on timeout
        }
    }

    let start = Instant::now();

    // Wait for echo LOW (pulse complete)
    let timeout = Instant::now();
    while echo.is_high() {
        if timeout.elapsed() > ECHO_TIMEOUT {
            defmt::warn!("Timeout waiting for LOW");
            return None; // Return early on timeout
        }
    }

    let end = Instant::now();

    // Calculate distance
    let time_elapsed = end.checked_duration_since(start)?.as_micros();
    let distance = time_elapsed as f64 * 0.0343 / 2.0;

    Some(distance)
}
```

We begin by setting the trigger pin low for a brief moment, then raising it high for 10 microseconds. This creates the trigger pulse that instructs the sensor to emit an ultrasonic burst. After that, we wait for the Echo pin to rise. The time the Echo pin stays high represents the round-trip travel time of the sound wave. Using this duration, we compute the final distance value and return it.

We have also added a timeout while waiting for the echo pin to change state so the code does not get stuck indefinitely. When the pin fails to respond within the allowed time, we treat the attempt as a failed reading and return None, which lets the rest of the program continue running normally.

## The main loop

Finally, let's create our main loop that continuously reads the sensor and updates the LED:

```rust
loop {
    Timer::after_millis(10).await;

    let distance = match measure_distance(&mut trigger, &echo).await {
        Some(d) => d,
        None => {
            Timer::after_secs(5).await;
            continue; // Skip to next iteration
        }
    };

    let duty_cycle = calculate_duty_cycle(distance, max_duty);
    led.set_duty_cycle(duty_cycle)
        .expect("duty cycle is within valid range");
}
```

Every 10 milliseconds, we measure the distance. If the measurement succeeds, we calculate the appropriate LED brightness and apply it. If it fails (due to timeout or sensor issues), we wait 5 seconds before trying again.

## The Full code

Here's everything put together:

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_rp as hal;
use embassy_rp::block::ImageDef;
use embassy_time::{Duration, Timer};

//Panic Handler
use panic_probe as _;
// Defmt Logging
use defmt_rtt as _;

// For GPIO
use embassy_rp::gpio::{Input, Level, Output, Pull};

// For PWM
use embassy_rp::pwm::{Pwm, SetDutyCycle};

// For time calculation
use embassy_time::Instant;

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());

    // For Onboard LED
    // let mut led = Pwm::new_output_b(p.PWM_SLICE4, p.PIN_25, Default::default());

    // For external LED connected on GPIO 3
    let mut led = Pwm::new_output_b(p.PWM_SLICE1, p.PIN_3, Default::default());

    let mut trigger = Output::new(p.PIN_17, Level::Low);
    let echo = Input::new(p.PIN_16, Pull::None);

    led.set_duty_cycle(0)
        .expect("duty cycle is within valid range");

    let max_duty = led.max_duty_cycle();
    // defmt::info!("Max duty cycle {}", max_duty);

    loop {
        Timer::after_millis(10).await;

        let distance = match measure_distance(&mut trigger, &echo).await {
            Some(d) => d,
            None => {
                Timer::after_secs(5).await;
                continue; // Skip to next iteration
            }
        };

        let duty_cycle = calculate_duty_cycle(distance, max_duty);
        led.set_duty_cycle(duty_cycle)
            .expect("duty cycle is within valid range");
    }
}

const ECHO_TIMEOUT: Duration = Duration::from_millis(100);

async fn measure_distance(trigger: &mut Output<'_>, echo: &Input<'_>) -> Option<f64> {
    // Send trigger pulse
    trigger.set_low();
    Timer::after_micros(2).await;
    trigger.set_high();
    Timer::after_micros(10).await;
    trigger.set_low();

    // Wait for echo HIGH (sensor responding)
    let timeout = Instant::now();
    while echo.is_low() {
        if timeout.elapsed() > ECHO_TIMEOUT {
            defmt::warn!("Timeout waiting for HIGH");
            return None; // Return early on timeout
        }
    }

    let start = Instant::now();

    // Wait for echo LOW (pulse complete)
    let timeout = Instant::now();
    while echo.is_high() {
        if timeout.elapsed() > ECHO_TIMEOUT {
            defmt::warn!("Timeout waiting for LOW");
            return None; // Return early on timeout
        }
    }

    let end = Instant::now();

    // Calculate distance
    let time_elapsed = end.checked_duration_since(start)?.as_micros();
    let distance = time_elapsed as f64 * 0.0343 / 2.0;

    Some(distance)
}

const MAX_DISTANCE_CM: f64 = 30.0;

fn calculate_duty_cycle(distance: f64, max_duty: u16) -> u16 {
    if distance < MAX_DISTANCE_CM && distance >= 2.0 {
        let normalized = (MAX_DISTANCE_CM - distance) / MAX_DISTANCE_CM;
        // defmt::info!("duty cycle :{}", (normalized * max_duty as f64) as u16);
        (normalized * max_duty as f64) as u16
    } else {
        0
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [embassy_rp::binary_info::EntryAddr; 4] = [
    embassy_rp::binary_info::rp_program_name!(c"ultrasonic"),
    embassy_rp::binary_info::rp_program_description!(c"your program description"),
    embassy_rp::binary_info::rp_cargo_version!(),
    embassy_rp::binary_info::rp_program_build_attribute!(),
];

// End of file
```

## Clone the existing project

You can clone (or refer) project I created and navigate to the `ultrasonic` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/ultrasonic
```

