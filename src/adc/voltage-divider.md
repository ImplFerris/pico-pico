# Voltage Divider with ADC

Sometimes you need to measure voltages higher than the ADC's reference voltage (3.3V). For example, measuring a 5V power supply, a 9V battery, or a 12V system. Since the RP2350 ADC can only safely measure up to 3.3V, you need to scale down the voltage using a voltage divider.

We have already seen how the voltage divider works and how two resistors can be used to scale a voltage. If you need a refresher on how voltage dividers work, refer back to the [voltage divider](../core-concepts/voltage-divider.md) section. Here, we will focus on how a voltage divider interacts with an ADC and how the divider output turns into an ADC reading.

> [!TIP]
> Voltage dividers are not just for scaling voltages.
> Sensors like LDRs and thermistors are resistors whose values change with light or temperature.
> When used in a voltage divider, these resistance changes turn into voltage changes that the ADC can read.

## Using Voltage Dividers with ADC

When using a voltage divider with an ADC, the goal is usually to scale a voltage so that it stays within the ADC input range.Design the voltage divider so that the maximum expected input voltage produces an output voltage below the ADC reference voltage.  

For a 3.3 V system, it is good practice to aim for 3.0 V or less to allow some margin for supply variation and noise.

## Example: Measuring a 3.3V Supply

Consider a simple example where the input voltage is 3.3V and we use resistor values of R1 = 10 kΩ and R2 = 1 kΩ.

You should already be familiar with the voltage divider formula:

\\[
V_{out} = V_{in} \times \frac{R_2}{R_1 + R_2}
\\]

Using the voltage divider equation:

\\[
V_{out} = 3.3V \times \frac{1k}{1k + 10k} = 3.3V \times \frac{1k}{11k} = 0.3V
\\]

This output voltage is well within the ADC input range. The divider output voltage `V_out` is connected to an ADC input pin.
