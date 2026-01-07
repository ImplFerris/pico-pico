# Servo Motor Control on Raspberry Pi Pico Using rp-hal

In this exercise, we repeat the same servo control example, but this time using rp hal instead of Embassy. The overall idea stays exactly the same. The main difference here is that we will use a fractional divider instead of a whole number divider.

For this, we will rely on the [calculator form](../core-concepts/pwm/frequency-to-pwm-top-divider.md) to generate the TOP value and both the integer and fractional parts of the divider.

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```

When prompted, give your project a name like "servo-motor" and choose "rp-hal" as the HAL. Enable defmt logging, if you have a debug probe so you can view logs also.

## Additional Imports

Along with the usual rp hal boilerplate, we need to bring in the trait that allows us to update the PWM duty cycle.

```rust
// For PWM
use embedded_hal::pwm::SetDutyCycle;
```

## Initialize PWM Slice

Next, we initialize the PWM peripheral and select the slice we want to use. In our case, we are using PWM slice 7 (since we are using GPIO 15).

```rust
let mut pwm_slices = hal::pwm::Slices::new(pac.PWM, &mut pac.RESETS);
let pwm = &mut pwm_slices.pwm7;
```

## Configure Divider and TOP

Now we apply the TOP value and the divider that were generated using the calculator form. This time, we explicitly set both the integer and fractional parts of the divider.

```rust
pwm.set_div_int(45);
pwm.set_div_frac(13);
pwm.set_top(65483);
pwm.enable();
```

## Attach PWM Channel to GPIO

```rust
let servo = &mut pwm.channel_b;
servo.output_to(pins.gpio15);
```

## Main Loop

Finally, inside the main loop, we update the duty cycle to move the servo between different positions. Just like in the Embassy example, we use set_duty_cycle_fraction.

```rust
loop {
    // Move servo to 0° position (2.5% duty cycle = 25/1000)
    servo
        .set_duty_cycle_fraction(25, 1000)
        .expect("invalid min duty cycle");
    timer.delay_ms(1000);

    // 90° position (7.5% duty cycle)
    servo
        .set_duty_cycle_fraction(75, 1000)
        .expect("invalid half duty cycle");
    timer.delay_ms(1000);

    // 180° position (12% duty cycle)
    servo
        .set_duty_cycle_fraction(120, 1000)
        .expect("invalid max duty cycle");
    timer.delay_ms(1000);
}
```

## Clone the existing project

You can clone (or refer) project I created and navigate to the `servo-motor` folder.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-projects/servo-motor
```

## The Full Code

```rust
#![no_std]
#![no_main]

use embedded_hal::delay::DelayNs;
use hal::block::ImageDef;
use rp235x_hal as hal;

use embedded_hal::pwm::SetDutyCycle;

//Panic Handler
use panic_probe as _;
// Defmt Logging
use defmt_rtt as _;

/// Tell the Boot ROM about our application
#[unsafe(link_section = ".start_block")]
#[used]
pub static IMAGE_DEF: ImageDef = hal::block::ImageDef::secure_exe();
/// External high-speed crystal on the Raspberry Pi Pico 2 board is 12 MHz.
/// Adjust if your board has a different frequency
const XTAL_FREQ_HZ: u32 = 12_000_000u32;

#[hal::entry]
fn main() -> ! {
    // Grab our singleton objects
    let mut pac = hal::pac::Peripherals::take().unwrap();

    // Set up the watchdog driver - needed by the clock setup code
    let mut watchdog = hal::Watchdog::new(pac.WATCHDOG);

    // Configure the clocks
    //
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

    let mut timer = hal::Timer::new_timer0(pac.TIMER0, &mut pac.RESETS, &clocks);

    let mut pwm_slices = hal::pwm::Slices::new(pac.PWM, &mut pac.RESETS);
    let pwm = &mut pwm_slices.pwm7;

    pwm.set_div_int(45);
    pwm.set_div_frac(13);
    pwm.set_top(65483);
    pwm.enable();

    let servo = &mut pwm.channel_b;
    servo.output_to(pins.gpio15);

    loop {
        // Move servo to 0° position (2.5% duty cycle = 25/1000)
        servo
            .set_duty_cycle_fraction(25, 1000)
            .expect("invalid min duty cycle");
        timer.delay_ms(1000);

        // 90° position (7.5% duty cycle)
        servo
            .set_duty_cycle_fraction(75, 1000)
            .expect("invalid half duty cycle");
        timer.delay_ms(1000);

        // 180° position (12% duty cycle)
        servo
            .set_duty_cycle_fraction(120, 1000)
            .expect("invalid max duty cycle");
        timer.delay_ms(1000);
    }
}

// Program metadata for `picotool info`.
// This isn't needed, but it's recomended to have these minimal entries.
#[unsafe(link_section = ".bi_entries")]
#[used]
pub static PICOTOOL_ENTRIES: [hal::binary_info::EntryAddr; 5] = [
    hal::binary_info::rp_cargo_bin_name!(),
    hal::binary_info::rp_cargo_version!(),
    hal::binary_info::rp_program_description!(c"your program description"),
    hal::binary_info::rp_cargo_homepage_url!(),
    hal::binary_info::rp_program_build_attribute!(),
];

// End of file
```
