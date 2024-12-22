## Action

Setting Up the PWM and Servo Control
```rust
const PWM_DIV_INT: u8 = 64;
const PWM_TOP: u16 = 46_874;

const TOP: u16 = PWM_TOP + 1;
const MIN_DUTY: u16 = (TOP as f64 * (2.5 / 100.)) as u16;
const HALF_DUTY: u16 = (TOP as f64 * (7.5 / 100.)) as u16;
const MAX_DUTY: u16 = (TOP as f64 * (12. / 100.)) as u16;
```


#### 1. Set Up the PWM Slice and Channel

First, initialize the PWM slice and channel. You should have already done similar in the previous blinky section.

```rust
let mut pwm_slices = hal::pwm::Slices::new(pac.PWM, &mut pac.RESETS);
let pwm = &mut pwm_slices.pwm4;
```

### 2. Adjust for 50HZ frequency
Now, set the divisor and the top value to achieve a PWM frequency of 50Hz.

```rust
pwm.set_div_int(PWM_DIV_INT);
pwm.set_div_frac(0);

pwm.set_top(PWM_TOP);
pwm.enable();
```

### 3. Set Output Pin
Next, specify the GPIO pin where the PWM signal will be sent. We will use GPIO pin 9.

```rust
    let servo = &mut pwm.channel_b;
    servo.output_to(pins.gpio9);

```
### 4. Set Servo Position in a Loop
Finally, in the loop, we adjust the duty cycle which will control the servo's position. We will move the servo to different positions (0°, 90°, and 180°) using the MIN_DUTY, HALF_DUTY, and MAX_DUTY values calculated earlier.

```rust
loop {
    servo.set_duty_cycle(MIN_DUTY).unwrap(); // 0 degrees
    timer.delay_ms(1000);

    servo.set_duty_cycle(HALF_DUTY).unwrap(); // 90 degrees
    timer.delay_ms(1000);

    servo.set_duty_cycle(MAX_DUTY).unwrap(); // 180 degrees
    timer.delay_ms(1000);
}

```

### Full Code snippet
```rust
const PWM_DIV_INT: u8 = 64;
const PWM_TOP: u16 = 46_874;

const TOP: u16 = PWM_TOP + 1;
const MIN_DUTY: u16 = (TOP as f64 * (2.5 / 100.)) as u16;
const HALF_DUTY: u16 = (TOP as f64 * (7.5 / 100.)) as u16;
const MAX_DUTY: u16 = (TOP as f64 * (12. / 100.)) as u16;

#[hal::entry]
fn main() -> ! {
    // Grab our singleton objects
    let mut pac = hal::pac::Peripherals::take().unwrap();

    // Set up the watchdog driver - needed by the clock setup code
    let mut watchdog = hal::Watchdog::new(pac.WATCHDOG);

    // Configure the clocks
    // The default is to generate a 125 MHz system clock
    let clocks = hal::clocks::init_clocks_and_plls(
        XTAL_FREQ_HZ,
        pac.XOSC,
        pac.CLOCKS,
        pac.PLL_SYS,
        pac.PLL_USB,
        &mut pac.RESETS,
        &mut watchdog,
    )
    .ok()
    .unwrap();

    // The single-cycle I/O block controls our GPIO pins
    let sio = hal::Sio::new(pac.SIO);

    // Set the pins up according to their function on this particular board
    let pins = hal::gpio::Pins::new(
        pac.IO_BANK0,
        pac.PADS_BANK0,
        sio.gpio_bank0,
        &mut pac.RESETS,
    );

    // The delay object lets us wait for specified amounts of time (in
    // milliseconds)
    let mut timer = hal::Timer::new_timer0(pac.TIMER0, &mut pac.RESETS, &clocks);

    // Init PWMs
    let mut pwm_slices = hal::pwm::Slices::new(pac.PWM, &mut pac.RESETS);

    // Configure PWM4
    let pwm = &mut pwm_slices.pwm4;

    pwm.set_div_int(PWM_DIV_INT);
    pwm.set_div_frac(0);

    pwm.set_top(PWM_TOP);
    pwm.enable();

    let servo = &mut pwm.channel_b;
    servo.output_to(pins.gpio9);

    loop {
        servo.set_duty_cycle(MIN_DUTY).unwrap();
        timer.delay_ms(1000);
        servo.set_duty_cycle(HALF_DUTY).unwrap();
        timer.delay_ms(1000);
        servo.set_duty_cycle(MAX_DUTY).unwrap();
        timer.delay_ms(1000);
    }
}
```

## Clone the existing project
You can clone (or refer) project I created and navigate to the `servo` folder.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-projects/servo
```
