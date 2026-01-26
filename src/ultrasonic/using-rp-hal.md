# Writing Rust Code Use HC-SR04 Ultrasonic Sensor with Pico 2

We'll start by generating the project using the template, then modify the code to fit the current project's requirements.

## Generating From template

Refer to the [Template section](../cargo-generate.md) for details and instructions.

To generate the project, run:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

When prompted, choose a name for your project-let's go with "bat-beacon". Don't forget to select `rp-hal` as the HAL.

Then, navigate into the project folder:

```sh
cd PROJECT_NAME
# For example, if you named your project "bat-beacon":
# cd bat-beacon
```

## Setup the LED Pin

You should understand this code by now. If not, please complete the Blink LED section first.

Quick recap: Here, we're configuring the PWM for the LED, which allows us to control the brightness by adjusting the duty cycle.

```rust
let pwm = &mut pwm_slices.pwm1;  // Access PWM slice 1
pwm.set_ph_correct();            // Set phase-correct mode for smoother transitions
pwm.enable();                    // Enable the PWM slice
let led = &mut pwm.channel_b; // Select PWM channel B
led.output_to(pins.gpio3);   // Set GPIO 3 as the PWM output pin
```

## Setup the Trigger Pin

The Trigger pin on the ultrasonic sensor is used to start the ultrasonic pulse. It needs to be set as an output so we can control it to send the pulse.

```rust
let mut trigger = pins.gpio17.into_push_pull_output();
```

## Setup the Echo Pin

The Echo pin on the ultrasonic sensor receives the returning signal, which allows us to measure the time it took for the pulse to travel to an object and back. It's set as an input to detect the returning pulse.

```rust
let mut echo = pins.gpio16.into_pull_down_input();
```

## 🦇 Light it Up

### Step 1: Send the Trigger Pulse

First, we need to send a short pulse to the trigger pin to start the ultrasonic measurement.

```rust
// Ensure the Trigger pin is low before starting
trigger.set_low().ok().unwrap();
timer.delay_us(2);

// Send a 10-microsecond high pulse
trigger.set_high().ok().unwrap();
timer.delay_us(10);
trigger.set_low().ok().unwrap();
```

### Step 2: Measure the Echo Time

Next, we will use two loops. The first loop will run as long as the echo pin state is LOW. Once it goes HIGH, we will record the current time in a variable. Then, we start the second loop, which will continue as long as the echo pin remains HIGH. When it returns to LOW, we will record the current time in another variable. The difference between these two times gives us the pulse width.

```rust
let mut time_low = 0;
let mut time_high = 0;

// Wait for the Echo pin to go high and note down the time
while echo.is_low().ok().unwrap() {
    time_low = timer.get_counter().ticks();
}

// Wait for the Echo pin to go low and note down the time
while echo.is_high().ok().unwrap() {
    time_high = timer.get_counter().ticks();
}

// Calculate the time taken for the signal to return
let time_passed = time_high - time_low;

```

### Step 3: Calculate Distance

To calculate the distance, we need to use the pulse width. The pulse width tells us how long it took for the ultrasonic waves to travel to an obstacle and return. Since the pulse represents the round-trip time, we divide it by 2 to account for the journey to the obstacle and back.

The speed of sound in air is approximately 0.0343 cm per microsecond. By multiplying the time (in microseconds) by this value and dividing by 2, we obtain the distance to the obstacle in centimeters.

```rust
let distance = time_passed as f64 * 0.0343 / 2.0;
```

### Step 4: PWM Duty cycle for LED

Finally, we adjust the LED brightness based on the measured distance.

The duty cycle percentage is calculated using our own logic, you can modify it to suit your needs. When the object is closer than 30 cm, the LED brightness will increase. The closer the object is to the ultrasonic module, the higher the calculated ratio will be, which in turn adjusts the duty cycle. This results in the LED brightness gradually increasing as the object approaches the sensor.

```rust
let duty_cycle = if distance < 30.0 {
    let step = 30.0 - distance;
    (step * 1500.) as u16 + 1000
} else {
    0
};

// Change the LED brightness
led.set_duty_cycle(duty_cycle).unwrap();
```

### Complete Logic of the loop

Note: This code snippet highlights the loop section and does not include the entire code.

```rust
loop {
    timer.delay_ms(5);

    trigger.set_low().ok().unwrap();
    timer.delay_us(2);
    trigger.set_high().ok().unwrap();
    timer.delay_us(10);
    trigger.set_low().ok().unwrap();

    let mut time_low = 0;
    let mut time_high = 0;
    while echo.is_low().ok().unwrap() {
        time_low = timer.get_counter().ticks();
    }
    while echo.is_high().ok().unwrap() {
        time_high = timer.get_counter().ticks();
    }
    let time_passed = time_high - time_low;

    let distance = time_passed as f64 * 0.0343 / 2.0;

    let duty_cycle = if distance < 30.0 {
        let step = 30.0 - distance;
        (step * 1500.) as u16 + 1000
    } else {
        0
    };
    led.set_duty_cycle(duty_cycle).unwrap();
}
```

## Clone the existing project

You can clone (or refer) project I created and navigate to the `ultrasonic` folder.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-rp-projects/ultrasonic
```

## Your Challenge

1. Use Embassy framework instead of `rp-hal`
2. Use the onboard LED instead
