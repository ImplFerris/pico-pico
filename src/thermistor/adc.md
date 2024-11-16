
## ADC
When setting up the thermistor with the Pico, we don't get the voltage directly. Instead, we receive an ADC value (refer to the [ADC](../ldr/adc.md) explanation in the LDR section). In the LDR exercise, we didn't calculate the voltage corresponding to the ADC value since we only needed to check whether the ADC value increased. However, in this exercise, to determine the temperature, we must convert the ADC value into voltage.


The formula is:

\\[
V_{\text{out}} = \text{ADC Value} \times \text{ADC Resolution}
\\]

Where:  
- **ADC Value**: The raw value read from the ADC.  
- **ADC Resolution**: The voltage resolution of the ADC; We have calculated 0.8mV for the Pico.


For example, let's take an ADC value of 2755:

\\[
\text{Voltage} = 2755 \times 0.8mV = 2204mV \approx 2.20V
\\]

