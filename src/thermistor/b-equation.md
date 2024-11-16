
## B Equation
The B equation is simpler but less precise.
\\[
\frac{1}{T} = \frac{1}{T_0} + \frac{1}{B} \ln \left( \frac{R}{R_0} \right)
\\]

Where:

- T is the temperature in **Kelvin**.
- \\( T_0 \\)  is the reference temperature (usually 298.15K or 25°C), where the thermistor's resistance is known (typically 10kΩ).
- R is the **resistance** at temperature T.
- \\( R_0 \\) is the **resistance** at the reference temperature \\( T_0 \\) (often 10kΩ).
- B is the **B-value** of the thermistor.

**Example Calculation:**

Given:
- Reference temperature \\( T_0 = 298.15K \\) (i.e., 25°C + 273.15 to convert to Kelvin)
- Reference resistance \\( R_0 = 10k\Omega \\)
- B-value B = 3950 (typical for many thermistors)
- Measured resistance at temperature T: 5kΩ

### Step 1: Apply the B-parameter equation
Substitute the given values:

\\[
\frac{1}{T} = \frac{1}{298.15} + \frac{1}{3950} \ln \left( \frac{5k}{10k} \right)
\\]

\\[
\frac{1}{T} = 0.003354016 + \frac{1}{3950} \ln(0.5)
\\]

\\[
\frac{1}{T} = 0.003354016 + (−0.000175481)
\\]

\\[
\frac{1}{T} = 0.003178535
\\]

### Step 2: Calculate the temperature (T)

\\[
T = \frac{1}{0.003178535} = 314.61034722K
\\]

Convert to Celsius:

\\[
T_{Celsius} = 314.61034722 - 273.15 \approx 41.46°C
\\]

### Result:

The temperature corresponding to a resistance of 5kΩ is approximately **41.46°C**.

### Rust function

```rust
/// Calculates the temperature in Kelvin based on the B-parameter equation.
///
/// # Arguments
/// * `r` - The measured resistance in ohms (e.g., 5kΩ).
/// * `r0` - The reference resistance at temperature `t0` in ohms (e.g., 10kΩ).
/// * `t0` - The reference temperature in Kelvin (typically 298.15K for 25°C).
/// * `b` - The B-value of the thermistor.
///
/// # Returns
/// The temperature in Kelvin.
fn calculate_temperature(r: f64, r0: f64, t0: f64, b: f64) -> f64 {
    let ln_value = (r / r0).ln();
    let inv_t = 1.0 / t0 + (1.0 / b) * ln_value;
    1.0 / inv_t
}

fn kelvin_to_celsius(kelvin: f64) -> f64 {
    kelvin -  273.15
}

fn celsius_to_kelvin(celsius: f64) -> f64 {
    celsius + 273.15
}

fn main() {
    let t0 = celsius_to_kelvin(25.0); // Reference temperature 25°C
    let r0 = 10_000.0; // Reference resistance in ohms (10kΩ)
    let b = 3950.0; // B-value
    let r = 5_000.0; // Measured resistance in ohms (5kΩ)
    
    let temperature_kelvin = calculate_temperature(r, r0, t0, b);
    let temperature_celsius = kelvin_to_celsius(temperature_kelvin);
    println!("Temperature: {:.2} °C", temperature_celsius);
}

```
