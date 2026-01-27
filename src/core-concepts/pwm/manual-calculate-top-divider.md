# Manually Calculate Top

In this section, we will manually derive the TOP value for a given PWM frequency. This method requires trying different divider values and checking whether the resulting TOP value falls within the valid range.

There is a better approach in the next section, where you can use either Rust code or a calculator form to compute both TOP and the divider automatically. For now, this manual method is useful because it helps build intuition about how the clock, divider, and TOP value relate to each other.

The TOP value must be within the range 0 to 65534. Although TOP is stored in a 16-bit unsigned register, setting it to 65535 prevents achieving a true 100 percent duty cycle. This is because the duty cycle compare register (CC) must be set to TOP + 1 to achieve a true 100 percent duty cycle, and CC itself is also only 16 bits wide. By keeping TOP at or below 65534, the value TOP + 1 still fits in the CC register, allowing the full 0 to 100 percent duty cycle range to be represented correctly.

To ensure TOP stays within this limit, we will choose divider values that are powers of two, such as 8, 16, 32, or 64. This approach does not work for every possible frequency. In some cases, you may need other integer values or even fractional dividers. To keep things simple, we will start with this approach.

As an example, we will calculate the values required to generate a 50 Hz PWM signal for a servo motor.

## PWM Frequency Formula

The RP2350 datasheet defines the PWM frequency as:

\\[
f_{PWM} = \frac{f_{sys}}{\text{period}} = \frac{f_{sys}}{(\text{TOP} + 1) \times (\text{CSR\_PH\_CORRECT} + 1) \times \left( \text{DIV\_INT} + \frac{\text{DIV\_FRAC}}{16} \right)}
\\]

Here's the derived formula to get the TOP for the target frequency:

\\[
\text{TOP} =
\frac{f_{sys}}
{f_{PWM} \times (\text{CSR\_PH\_CORRECT} + 1) \times \left( \text{DIV\_INT} + \frac{\text{DIV\_FRAC}}{16} \right)} - 1
\\]

Where:
- \\( f_{PWM} \\) is the desired PWM frequency.
- \\( f_{sys} \\) is the system clock frequency. For the Pico 2, it is is 150 MHz.

We're not going to use phase correct mode and we're not using fraction for the divider either, so let's simplify the formula further:

\\[
\text{TOP} =
\frac{f_{sys}}
{f_{PWM} \times \text{DIV\_INT}} - 1
\\]

### TOP for 50 Hz

We want the PWM frequency to be 50 Hz. To achieve that, we substitute the system clock frequency, target frequency and the chosen divider integer, and we get the following TOP value:

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

You can experiment with different divider values (even including fraction) and corresponding top values.
