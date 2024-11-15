## Servo and Pico

To control a servo with the Raspberry Pi Pico, we need to set a 50Hz PWM frequency. Currently, RP-HAL doesn't allow directly setting the frequency, so we achieve this by adjusting the `top` and `div_int` values.

Refer the 1073th page of the [RP2350](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf) Datasheet to understand how `top` and `div_int` works.

### Formula from datasheet
The following formula from the datasheet is used to calculate the period and determine the output frequency based on the system clock frequency.

1. **Period calculation:**
\\[
\text{period} = (\text{TOP} + 1) \times (\text{CSR\_PH\_CORRECT} + 1) \times \left( \text{DIV\_INT} + \frac{\text{DIV\_FRAC}}{16} \right)
\\]

2. **PWM output frequency calculation:**

\\[
f_{PWM} = \frac{f_{sys}}{\text{period}} = \frac{f_{sys}}{(\text{TOP} + 1) \times (\text{CSR\_PH\_CORRECT} + 1) \times \left( \text{DIV\_INT} + \frac{\text{DIV\_FRAC}}{16} \right)}
\\]

Where:
- \\( f_{PWM} \\) is the PWM output frequency.
- \\( f_{sys} \\) is the system clock frequency. For the pico2, it is is 150MHZ.

### Let's calculate `top`
We want the PWM frequency (f_pwm) to be 50 Hz. In order to achieve that, we are going to adjust the `top` and `div_int` values.

The top value must be within the range of 0 to 65535 (since it's a 16-bit unsigned integer). To make sure the top value fits within this range, I chose values for the divisor (div_int) in powers of 2 (such as 8, 16, 32, 64), though this isn't strictly necessary (it's just a preference). In this case, we chose `div_int = 64` to calculate a top value that fits within the u16 range.

With the chosen div_int and system parameters, we can calculate the top using the following formula:
\\[
\text{top} = \frac{150,000,000}{50 \times 64} - 1
\\]

\\[
\text{top} = \frac{150,000,000}{3,200} - 1
\\]

\\[
\text{top} = 46,875 - 1
\\]

\\[
\text{top} = 46,874
\\]

After performing the calculation, we find that the top value is `46,874`.

You can experiment with different div_int and corresponding top values. Just ensure that div_int stays within the u8 range, top fits within the u16 range, and the formula yields a 50Hz frequency.

Note:
- In case you are wondering, we are not setting the `div_frac` which is 0 by default. That's why it is not included in the calculation.
- We are not going to enable the phase correct for this exercise, so it also can be excluded from the calculation (since it is just multiplying by 1); if you enable phase correct, then the calculation will differ since you have to multiply by 2 (1+1)


### Position calculation based on top
To calculate the duty cycle that corresponds to specific positions (0, 90, and 180 degrees), we use the following formula based on the top value:

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

We multiply the TOP value by a duty cycle percentage to determine the appropriate pulse width for each position of the servo. You might need to adjust the percentage based on your servo.

