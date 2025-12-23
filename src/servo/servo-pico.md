## Position and Duty Cycle

To control a servo with the Raspberry Pi Pico, we need to set a 50 Hz PWM frequency. There is no straightforward way to set the frequency directly in embassy or rp-hal, at least to my knowledge.

In embassy-rp, PWM is configured using a `Config` struct with multiple fields. For our use case, we mainly care about the top and divider values. The same applies to rp-hal, where we can set top, div_int, and div_frac separately.

You can either use the [manual method](/core-concepts/pwm/manual-calculate-top-divider.md) to find a suitable TOP value, or [use the form](/core-concepts/pwm/frequency-to-pwm-top-divider.md) to automatically calculate both TOP and the divider for the target frequency of 50 Hz.

Using the manual method, I calculated a TOP value of 46,874 with a divider of 64. Using the form, I got a divider of 45.8125 with a TOP value of 65,483. We can use either of these configurations.

> [!NOTE]
> In rp-hal, you have to set the divider integer and fraction separately. So a divider of 64 becomes div_int = 64 and div_frac = 0. A divider of 45.8125 becomes div_int = 45 and div_frac = 13.  

## Position calculation based on top

Once the TOP value for a 50 Hz PWM signal is known, we can calculate the duty cycle values required to position the servo.

The servo determines its position by measuring the pulse width, which is the amount of time the signal stays high during each 20 ms PWM cycle. The exact pulse widths are not identical for all servos and can vary slightly depending on the specific servo model. 

In my case, the values were:

- 0° at about 0.5 ms, which corresponds to a 2.5% duty cycle since 0.5 ms is 2.5% of a 20 ms period.

- 90° at about 1.5 ms, which corresponds to a 7.5% duty cycle since 1.5 ms is 7.5% of a 20 ms period.

- 180° at about 2.4 ms, which corresponds to a 12% duty cycle since 2.4 ms is 12% of a 20 ms period.

In the LED dimming chapter, changing the duty cycle was straightforward. We only cared about brightness, not frequency, so using `set_duty_cycle_percent` was sufficient. That function accepts a u8 value from 0 to 100, which works well for whole-number percentages.

For servo control, this approach is not suitable because the required duty cycles include fractional values such as 2.5%, 7.5%, and 12%.

We therefore have two alternatives. One option is to calculate the duty value directly from TOP and use `set_duty_cycle`, which accepts a u16. The other option is to use `set_duty_cycle_fraction`, which lets you specify the duty cycle as a numerator and denominator.

### Option 1: Manual calculation with set_duty_cycle

We first convert the pulse width into a percentage of the period. That percentage is then multiplied by TOP + 1 to obtain the duty value that configures the PWM output.

```rust
const PWM_DIV_INT: u8 = 64;
const PWM_TOP: u16 = 46_874;

const TOP: u16 = PWM_TOP + 1;
// 0.5ms is 2.5% of 20ms; 0 degrees in servo
const MIN_DUTY: u16 = (TOP as f64 * (2.5 / 100.)) as u16; 
// 1.5ms is 7.5% of 20ms; 90 degrees in servo
const HALF_DUTY: u16 = (TOP as f64 * (7.5 / 100.)) as u16; 
// 2.4ms is 12% of 20ms; 180 degree in servo
const MAX_DUTY: u16 = (TOP as f64 * (12. / 100.)) as u16;
```
Once the duty value is calculated, it can be applied like this:

```rust
servo.set_duty_cycle(MIN_DUTY)
            .expect("invalid min duty cycle");
```

### Option 2: Using set_duty_cycle_fraction

Another option is to use `set_duty_cycle_fraction`. This will help us to set percentage with fraction. 

In fact, set_duty_cycle_percent is a convenience method provided by embedded-hal that internally calls set_duty_cycle_fraction. It simply divides the input percentage by 100 and forwards the result as a fraction.

From embedded-hal:
  
```rust
 /// Set the duty cycle to `percent / 100`
///
/// The caller is responsible for ensuring that `percent` is less than or equal to 100.
#[inline]
fn set_duty_cycle_percent(&mut self, percent: u8) -> Result<(), Self::Error> {
    self.set_duty_cycle_fraction(u16::from(percent), 100)
}

/// Set the duty cycle to `num / denom`.
///
/// The caller is responsible for ensuring that `num` is less than or equal to `denom`,
/// and that `denom` is not zero.
fn set_duty_cycle_fraction(&mut self, num: u16, denom: u16) -> Result<(), Self::Error> {
    debug_assert!(denom != 0);
    debug_assert!(num <= denom);
    let duty = u32::from(num) * u32::from(self.max_duty_cycle()) / u32::from(denom);

    // This is safe because we know that `num <= denom`, so `duty <= self.max_duty_cycle()` (u16)
    #[allow(clippy::cast_possible_truncation)]
    {
        self.set_duty_cycle(duty as u16)
    }
}
```

This function does not accept floating-point values. Instead, it takes a numerator and a denominator, both as u16. To represent fractional percentages, we simply scale them into integers.

Remember that 2.5% can be written as the fraction 2.5/100. Since we can't use decimals in the numerator, we multiply both the numerator and denominator by 10 to get equivalent integer fractions:

```rust
2.5/100 = (2.5 × 10)/(100 × 10) = 25/1000
```

Now we have an equivalent fraction using only integers. We can apply the same conversion to our other percentages:

For example:
- 2.5% can be written as 25 / 1000  (in other words, 25 is 2.5% of 1000)
- 7.5% can be written as 75 / 1000  (in other words, 75 is 7.5% of 1000)
- 12% can be written as 120 / 1000  (in other words, 120 is 12% of 1000)
 
So in our code, we can apply it like this:

```rust
// Move servo to 0° position (2.5% duty cycle = 25/1000)
servo.set_duty_cycle_fraction(25, 1000)
    .expect("invalid duty cycle");

// 90° position (7.5% duty cycle)
servo.set_duty_cycle_fraction(75, 1000)
    .expect("invalid duty cycle");

// 180° position (12% duty cycle)
servo.set_duty_cycle_fraction(120, 1000)
    .expect("invalid duty cycle");
```
