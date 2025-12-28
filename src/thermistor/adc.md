
## ADC
When setting up the thermistor with the Pico, we don't get the voltage directly. Instead, we receive an ADC value (refer to the [ADC](../core-concepts/adc.md) explanation in the LDR section). In the LDR exercise, we didn't calculate the resistance corresponding to the ADC value since we only needed to check whether the ADC value increased. However, in this exercise, to determine the temperature, we must convert the ADC value into resistence.

### ADC to Resistance
We need resistance value from the adc value for the thermistor temperature calculation(that will be discussed in the next chapters).

We will use this formula to calculate the resistance value from the ADC reading. If you need how it is derived, refer the [Deriving Resistance from ADC Value](./adc-maths.md).

\\[
R_2 = \frac{R_1}{\left( \frac{\text{ADC_MAX}}{\text{adc_value}} - 1 \right)}
\\]

Note: If you connected the thermistor to power supply instead of GND. You will need opposite. since thermistor becomes R1.

\\[
R_1 = {R_2} \times \left(\frac{\text{ADC_MAX}}{\text{adc_value}} - 1\right)
\\]


Where:  
- R2: The resistance based on the ADC value.  
- R1: Reference resistor value (typically 10kΩ)
- ADC_MAX: The maximum ADC value is 4095 (\\( 2^{12}\\) -1 ) for a 12-bit ADC
- adc_value: ADC reading (a value between 0 and ADC_MAX).



## From ADC Reading to Sensor Resistance

For simple sensors like an LDR, you can just use the ADC value itself and take action based on it. But when you are working with sensors like thermistors, you will need to calculate the sensor resistance first, then use that to derive the corresponding temperature value.

There are two common ways to wire a sensor in a voltage divider:

<div class="image-with-caption" style="text-align:center; ">
    <img src="./images/adc-voltage-divider-resistor-sensor.svg" alt="ADC Voltage Divider Resistor Sensor" style="height:auto; display:block; margin:0 auto;"/>
    <div class="caption" style="font-size:0.9em; color:#555; margin-top:6px;">ADC Voltage Divider Resistor Sensor</div>
</div>

In both cases, the other resistor is a fixed resistor, typically something like 10 kΩ.

### Calculating Sensor Resistance

First, convert your ADC reading back to voltage:

\\[
V_{out} = \frac{ADC_{reading}}{4096} \times V_{ref}
\\]

Then, depending on your configuration:

**If sensor is R2 (bottom position):**

\\[
R_{sensor} = R_1 \times \frac{V_{out}}{V_{in} - V_{out}}
\\]

**If sensor is R1 (top position):**

\\[
R_{sensor} = R_2 \times \frac{V_{in} - V_{out}}{V_{out}}
\\]

Where:
- `R_sensor` is the resistance you want to find
- `R_1` or `R_2` is your known fixed resistor value
- `V_in` is your supply voltage (usually 3.3V)
- `V_out` is the voltage you calculated from the ADC reading

Once you have the sensor resistance, you can use the sensor's datasheet or calibration data to convert resistance to the measured value (like temperature for a thermistor or light level for an LDR).



### Rust Function

```rust

const ADC_MAX: u16 = 4095;
const REF_RES: f64 = 10_000.0; 

fn adc_to_resistance(adc_value: u16, ref_res:f64) -> f64 {
    let x: f64 = (ADC_MAX as f64/adc_value as f64)  - 1.0;
    // ref_res * x // If you connected thermistor to power supply
    ref_res / x
}

fn main() {
    let adc_value = 2000; // Our example ADC value;

    let r2 = adc_to_resistance(adc_value, REF_RES);
    println!("Calculated Resistance (R2): {} Ω", r2);
}
```

