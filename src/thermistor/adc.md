
## ADC
When setting up the thermistor with the Pico, we don't get the voltage directly. Instead, we receive an ADC value (refer to the [ADC](../ldr/adc.md) explanation in the LDR section). In the LDR exercise, we didn't calculate the resistance corresponding to the ADC value since we only needed to check whether the ADC value increased. However, in this exercise, to determine the temperature, we must convert the ADC value into resistence.


<!-- ### ADC to Voltage
The formula to convert an ADC value to voltage is: -->

<!-- \\[
V_{\text{out}} = \text{ADC Value} \times \text{ADC Resolution}
\\] -->

<!-- \\[
{\text{v_out}} = \text{adc\_count} \times \frac{{\text{v_in}}}{\text{adc_max}} -->
\\]

<!-- Where:  
- **ADC Value**: The raw value read from the ADC.  
- **v_in**: The reference input voltage (3.3V for the Pico).
- **adc_max**: The maximum ADC value is 4095 (\\( 2^{12}\\) -1 ) for a 12-bit ADC. -->


<!-- For example, let's take an ADC value of 2000:

\\[
\text{Voltage} = 2000 \times \frac{{\text{3.3}}}{\text{4095}}  = 2000 \times 0.8mV = 1600mV \approx 1.6V
\\] -->

### ADC to Resistance
We need resistance value from the adc value for the thermistor temperature calculation(that will be discussed in the next chapters).

We will use this formula to calculate the resistance value from the ADC reading. I found this formula from Wokwi and a few other code examples. However, I am unable to correlate it with other formulas or fully understand why this particular formula is used. If you have any resources explaining the derivation of this formula, please submit a pull request on GitHub.

\\[
R_2 = \text{ref_res} \times \left(\frac{\text{ADC_MAX}}{\text{adc_count}} - 1\right)
\\]

Where:  
- R2: The resistance based on the ADC value.  
- ref_res: Reference resistor value (typically 10kΩ)
- ADC_MAX: The maximum ADC value is 4095 (\\( 2^{12}\\) -1 ) for a 12-bit ADC
- adc_count: ADC reading (a value between 0 and ADC_MAX).


### Rust Function

```rust

const ADC_MAX: u16 = 4095;
const REF_RES: f64 = 10_000.0; 

fn adc_to_resistance(adc_count: u16, ref_res:f64) -> f64 {
    let x: f64 = (ADC_MAX as f64/adc_count as f64)  - 1.0;
    ref_res * x
}

fn main() {
    let adc_count = 2000; // Our example ADC value;

    let r2 = adc_to_resistance(adc_count, REF_RES);
    println!("Calculated Resistance (R2): {} Ω", r2);
}
```

