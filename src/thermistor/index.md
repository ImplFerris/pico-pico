# Thermistor

A thermistor is a resistor whose value changes with temperature. As temperature goes up or down, the resistance changes in a predictable way.  The name "thermistor" is derived from a combination of the words "thermal" and "resistor."

From the microcontroller point of view, a thermistor is just another resistor connected to an ADC pin. The ADC does not know anything about temperature. It only measures voltage. We do the rest in the code.

## NTC vs PTC

There are two common types of thermistors.

An NTC thermistor has lower resistance at higher temperatures. This is the most common type used for temperature measurement in embedded systems.

<img style="display: block; margin: auto;" alt="pico2" src="./images/ntc-resistor.png"/>


A PTC thermistor has higher resistance at higher temperatures. These are usually used for protection or current limiting rather than precise sensing.

In this chapter, we are going to work with an NTC thermistor.


## Reference
- [Thermistor Basics](https://www.teamwavelength.com/thermistor-basics/)
- [Thermistors](https://www.electronics-tutorials.ws/io/thermistors.html)
