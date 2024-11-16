
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
- Measured resistance at temperature T: 10475Ω

### Step 1: Apply the B-parameter equation
Substitute the given values:

\\[
\frac{1}{T} = \frac{1}{298.15} + \frac{1}{3950} \ln \left( \frac{10,475}{10,000} \right)
\\]

\\[
\frac{1}{T} = 0.003354016 + \frac{1}{3950} \ln(1.0475)
\\]

\\[
\frac{1}{T} = 0.003354016 + (0.000011748)
\\]

\\[
\frac{1}{T} = 0.003365764
\\]

### Step 2: Calculate the temperature (T)

\\[
T = \frac{1}{0.003365764} = 297.10936358 (Kelvin)
\\]

Convert to Celsius:

\\[
T_{Celsius} = 297.10936358 - 273.15 \approx 23.95936358°C
\\]

### Result:

The temperature corresponding to a resistance of 10475Ω is approximately **23.96°C**.

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
const REF_RES: f64 = 10_000.0; // Reference resistance in ohms (10kΩ)
const REF_TEMP: f64 = 25.0;  // Reference temperature 25°C

fn main() {
    let t0 = celsius_to_kelvin(REF_TEMP);
    let r = 10_475.0; // Measured resistance in ohms
    
    let temperature_kelvin = calculate_temperature(r, REF_RES, t0, B_VALUE);
    let temperature_celsius = kelvin_to_celsius(temperature_kelvin);
    println!("Temperature: {:.2} °C", temperature_celsius);
}

```
