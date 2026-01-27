# ADC (Analog to Digital Converter)

I hope you are already familiar with the idea of analog and digital signals. The world around us is mostly analog. Temperature rises and falls smoothly, light intensity changes gradually, sound moves through the air as waves, and sensors produce voltages that change continuously.

Microcontrollers, however, only understand discrete values. Internally, everything is represented as binary numbers, ones and zeros, HIGH and LOW. This creates a basic gap between the real world and digital logic. How does a digital system understand something that changes smoothly?

This is where the Analog to Digital Converter(ADC) comes in.  An Analog to Digital Converter (ADC) measures an analog voltage and converts it into a digital number that our microcontrollers can process, store, and act upon.

You may recall from the earlier chapter on PWM that we explored how a digital system can create an analog-like output by rapidly switching a pin on and off. ADC does the opposite. It allows the microcontroller to read and understand analog inputs from the real world. Together, peripherals like PWM and ADC allow embedded systems to both sense their environment and control it.

Consider a simple example: a temperature monitoring system. A temperature sensor outputs a voltage that changes with temperature, for example 0 V at 0 °C and 3.3 V at 100 °C. Without an ADC, this smooth voltage signal would be meaningless to the microcontroller. The ADC samples the voltage at regular intervals and converts each sample into a digital value. Your code can then read that value, interpret it as a temperature, and respond accordingly.

<img style="display: block; margin: auto;" alt="pico2" src="./images/adc.jpg"/>

## ADC Resolution

One of the most important characteristics of an ADC is its resolution, which determines how finely it can divide the analog input range into discrete digital values. Resolution is typically expressed in bits, such as 8-bit, 10-bit, or 12-bit. The number of bits decides how many different values the ADC can produce.

An 8-bit ADC divides its input voltage range into 256 distinct levels (2<sup>8</sup>). A 10-bit ADC provides 1,024 levels (2<sup>10</sup>), and a 12-bit ADC offers 4,096 levels(2<sup>12</sup>). As you can see, each additional bit doubles the number of levels, significantly improving precision. The more bits your ADC has, the smaller the voltage differences it can distinguish.

### Example

Let's make this concrete with an example. Suppose you are using a 10-bit ADC with a reference voltage of 3.3 V. The ADC will represent voltages from 0 V to 3.3 V as digital values from 0 to 1,023.

The smallest voltage change the ADC can detect (called the step size) is calculated as:

\\[
\text{Step Size} = \frac{\text{Vref}}{2^{\text{bits}}}
\\]

```text
Step Size = 3.3 V / 1,024 = 3.22 mV
```

This means the ADC cannot tell the difference between two voltages that are closer than about 3.22 mV. For example, if a sensor output changes from 1.500 V to 1.502 V, the ADC will likely report the same digital value for both.

Compare this to a 12-bit ADC with the same 3.3 V reference:

```text
Step Size = 3.3 V / 4,096 = 0.81 mV
```

The 12-bit converter offers significantly finer precision, which can be critical in applications requiring accurate measurements.

## ADC Formula

Conceptually, an ADC converts an input voltage into a digital number using the following relationship:

\\[
ADC = \frac{V_{in}}{V_{ref}} \times 2^{\text{bits}}
\\]

Where:
- \\(V_{in}\\) is the input voltage
- \\(V_{ref}\\) is the reference voltage

## Converting ADC Values to Voltage

When you read from the ADC in your code, it returns a raw digital value between 0 and 4095 (for a 12-bit ADC). This number represents where the measured voltage falls within the 0 V to 3.3 V range. To convert this raw value back into an actual voltage, you need a simple calculation.

The conversion formula is:

\\[
\text{Voltage} = \frac{\text{ADC Reading} \times \text{Reference Voltage}}{2^{\text{bits}}}
\\]

For the RP2350 with its 12-bit ADC and 3.3 V reference voltage:

\\[
\text{Voltage} = \frac{\text{ADC Reading} \times 3.3}{4096}
\\]

### Example

Let's say we get an ADC reading of 1000.

```text
Voltage = (1000 × 3.3) / 4096 = 0.806 V
```

## Reference

- [What is Analog to Digital Converter & Its Working](https://www.elprocus.com/analog-to-digital-converter/)
