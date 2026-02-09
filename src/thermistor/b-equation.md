
# B Equation

The B equation, also called the Beta parameter method, is the simplest way to convert thermistor resistance into temperature. It uses a single material constant provided by the manufacturer to approximate the thermistor's behavior.

## The B equation formula

\\[
\frac{1}{T} = \frac{1}{T_0} + \frac{1}{B} \ln \left( \frac{R}{R_0} \right)
\\]

In this equation, T is the temperature in Kelvin that we want to find. R is the measured resistance at the unknown temperature T.

\\( T_0 \\) is the reference temperature, usually 298.15 K (25 °C), where the thermistor's resistance is known.  \\( R_0 \\) is the resistance at the reference temperature \\( T_0 \\), often 10 kΩ for common thermistors.

B is the B-value of the thermistor, a material constant. And, the ln represents the natural logarithm function.

### Understanding the parameters

**R₀ (Reference resistance)**:

This is the thermistor's resistance at a known temperature, usually 25 °C. For example, a "10 kΩ thermistor" has R₀ = 10,000 ohms at 25 °C. This value should be specified in the datasheet.

**B (Beta value)**:

The B value is a constant usually provided by the manufacturers. The value describes how quickly resistance changes with temperature. Common values range from 3000 to 4000 K.

The datasheet often specifies B over a temperature range, such as B₂₅/₈₅ = 3950, meaning the Beta value is 3950 between 25 °C and 85 °C.

**Temperature in Kelvin**:

The temperature in the B equation must be in Kelvin (Kelvin = Celsius + 273.15), not Celsius. Convert to Kelvin before using the equation, and subtract 273.15 from the result to get Celsius again.

## Example Calculation

Let us say the measured resistance of the thermistor is 10,475 Ω, which corresponds to R in the equation. To calculate the temperature, we substitute this value along with the reference temperature \\( T_0 = 298.15 K \\) (25 °C), the reference resistance \\( R_0 = 10 k\Omega \\), and the thermistor B-value of 3950 into the B equation.

**Step 1**: Calculate the resistance ratio
\\[
\frac{R}{R_0} = \frac{10475}{10000} = 1.0475
\\]

**Step 2**: Calculate the natural logarithm of the ratio
\\[
\ln(1.0475) = 0.04641
\\]

**Step 3**: Apply the B equation
\\[
\frac{1}{T} = \frac{1}{298.15} + \frac{1}{3950} \times 0.04641
\\]

\\[
\frac{1}{T} = 0.003354 + 0.00001175 = 0.003366
\\]

**Step 4**: Calculate T by taking the reciprocal
\\[
T = \frac{1}{0.003366} = 297.1 K
\\]

**Step 5**: Convert to Celsius
\\[
T = 297.1 - 273.15 = 23.95 °C
\\]

The measured resistance of 10,475 Ω corresponds to a temperature of approximately 24 °C.

### Rust function

```rust
fn calculate_temperature(current_res: f64, ref_res: f64, ref_temp: f64, b_val: f64) -> f64 {
    let ln_value = (current_res/ref_res).ln();
    // let ln_value = libm::log(current_res / ref_res); // use this crate for no_std
    let inv_t = (1.0 / ref_temp) + ((1.0 / b_val) * ln_value);
    1.0 / inv_t
}

fn kelvin_to_celsius(kelvin: f64) -> f64 {
    kelvin -  273.15
}

fn celsius_to_kelvin(celsius: f64) -> f64 {
    celsius + 273.15
}

const B_VALUE: f64 = 3950.0;
const V_IN: f64 = 3.3; // Input voltage
const REF_RES: f64 = 10_000.0; // Reference resistance in ohms (10 kΩ)
const REF_TEMP: f64 = 25.0;  // Reference temperature 25 °C

fn main() {
    let t0 = celsius_to_kelvin(REF_TEMP);
    let r = 10475.0; // Measured resistance in ohms

    let temperature_kelvin = calculate_temperature(r, REF_RES, t0, B_VALUE);
    let temperature_celsius = kelvin_to_celsius(temperature_kelvin);
    println!("Temperature: {:.2} °C", temperature_celsius);
}

```
