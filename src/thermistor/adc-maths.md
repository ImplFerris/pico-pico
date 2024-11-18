## Derivations

You can skip this section if you'd like. It simply explains the math behind deriving the resistance from the ADC value.

### ADC to Voltage
The formula to convert an ADC value to voltage is: 

 \\[
V_{\text{out}} = {{V_{in}}}  \times \frac{\text{adc_value}}{\text{adc_max}}
\\]

Where:  
- **adc_value**: The value read from the ADC.  
- **v_in**: The reference input voltage (3.3V for the Pico).
- **adc_max**: The maximum ADC value is 4095 (\\( 2^{12}\\) -1 ) for a 12-bit ADC.

### Deriving Resistance from ADC Value

We combine the voltage divider formula with ADC Resolution formula to find the Resistance(R2). 

<span style="color:orange">Note:</span> It is assumed here that one side of the thermistor is connected to Ground (GND). I noticed that some online articles do the opposite, connecting one side of the thermistor to the power supply instead, which initially caused me some confusion.
<!-- 
**ADC voltage calculation formula**
\\[
V_{out} = {{V_{in}}} \times \frac{\text{adc_value}}{\text{adc_max}}
\\] -->

**Votlage Divider Formula**
\\[
V_{out} = V_{in} \times \frac{R_2}{R_1 + R_2}
\\]


#### Step 1:
We can substitue the Vout and make derive it like this

\\[
{V_{in}} \times \frac{\text{adc_value}}{\text{adc_max}} = V_{in} \times \frac{R_2}{R_1 + R_2}
\\]

\\[
\require{cancel}
\cancel{V_{in}} \times \frac{\text{adc_value}}{\text{adc_max}} = \cancel {V_{in}} \times \frac{R_2}{R_1 + R_2}
\\]


#### Step 2:
Lets temperoarily assign the adc_value/adc_max to x for ease of derivation and finally subsitue

\\[
x = \frac{\text{adc_value}}{\text{adc\_max}}
\\]

Substituting x into the equation:

\\[
x = \frac{R_2}{R_1 + R_2}
\\]

Rearrange to Solve \\( R_2 \\)

\\[
R_2 = x \times (R_1 + R_2)
\\]

Expand the right-hand side:

\\[
R_2 = x \times R_1 + x \times R_2
\\]

Rearrange to isolate \\( R_2 \\) terms:

\\[
R_2 - x \times R_2 = R_1 \times x
\\]


\\[
R_2 \times (1 - x) =  R_1 \times x
\\]


\\[
R_2 = R_1  \times  \frac{{x}}{{1-x}} 
\\]


\\[
R_2 = R_1 \times \frac{1}{\left( \frac{1}{x} - 1 \right)}
\\]

#### Step 3
 
Let's subsitute the x value back. We need 1/x, lets convert it. 
\\[
\frac{1}{x} = \frac{\text{adc\_max}}{\text{adc_value}}
\\]

---

#### Final Formula

\\[
R_2 = R_1 \times \frac{1}{\left( \frac{\text{adc\_max}}{\text{adc_value}} - 1 \right)}
\\]

