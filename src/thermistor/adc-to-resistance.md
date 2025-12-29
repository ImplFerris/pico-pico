
# ADC to Resistance

When we connect a thermistor to the Pico, we do not read the voltage directly. What we get from the ADC is just a digital value that represents that voltage (refer to the [ADC](../adc/index.md) chapter).

But to calculate temperature, we actually need the resistance of the thermistor at that moment, not the ADC value itself.

In the next chapter, we are going to look at the temperature equations. Before we can use those formulas, we first need to convert the ADC reading back into the thermistor resistance. That is exactly what this chapter focuses on.

## Converting ADC Reading to Thermistor Resistance

We now derive the thermistor resistance directly from the ADC reading using the voltage divider equation and the ADC conversion equation.

### Step 1: Voltage at the divider midpoint

This is the standard voltage divider formula we have already seen. Since the thermistor is connected as R1, we label it explicitly as the thermistor resistance.

\\[
V_{in} = \\frac{R_2}{R_{thermistor} + R_2} \\times V_{ref}
\\]


### Step 2: ADC conversion equation

The ADC converts the input voltage into a digital value based on the ratio of the input voltage to the reference voltage.

\\[
ADC = \\frac{V_{in}}{V_{ref}} \\times 2^{bits}
\\]


### Step 3: Substitute the divider equation into the ADC equation

Substituting the expression for \\( V_{in} \\) into the ADC equation:

\\[
ADC = \\frac{\\frac{R_2}{R_{thermistor} + R_2} \\times V_{ref}}{V_{ref}} \\times 2^{bits}
\\]

The \\( V_{ref} \\) terms cancel out, leaving:

\\[
ADC = \\frac{R_2}{R_{thermistor} + R_2} \\times 2^{bits}
\\]

### Step 4: Solve for thermistor resistance

Rearranging and solving for \\( R_{thermistor} \\):

\\[
R_{thermistor} = \\left( \\frac{2^{bits}}{ADC} - 1 \\right) \\times R_2
\\]

For the Pico, since it uses a 12-bit ADC, the equation can be simplified to:

\\[
R_{thermistor} = \\left( \\frac{4096}{ADC} - 1 \\right) \\times R_2
\\]

At this point, the equation contains only the ADC result, which is measured, and \\( R_2 \\), which is a known resistor value. Because \\( R_2 \\) directly affects the calculated thermistor resistance, it should be a temperature-stable resistor. Any variation in \\( R_2 \\) with temperature will directly affect the accuracy of the measurement.

In the next chapter, we will use this resistance value to calculate the temperature.

### Rust Function

A simple helper function that converts an ADC reading into the corresponding thermistor resistance.

```rust
const ADC_LEVELS: f64 = 4096.0;
const R2_RES: f64 = 10_000.0; // R2

fn adc_to_resistance(adc_value: u16, r2_res:f64) -> f64 {
     let adc = adc_value as f64;
    ((ADC_LEVELS / adc) - 1.0) * r2_res
}

fn main() {
    let adc_value = 2000; // Our example ADC value;

    let r2 = adc_to_resistance(adc_value, R2_RES);
    println!("Calculated Resistance (R2): {} Ω", r2);
}
```

