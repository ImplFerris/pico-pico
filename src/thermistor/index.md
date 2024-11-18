# Thermistor

In this section, we'll be using a thermistor with the Raspberry Pi Pico.  A thermistor is a variable resistor that changes its resistance based on the temperature. The amount of change in resistance depends on its composition.  The term comes from combining "thermal" and "resistor.". 

Thermistors are categorized into two types:
- NTC (Negative Temperature Coefficient): 
    - Resistance decreases as temperature increases. 
    - They are primarily used for temperature sensing and inrush current limiting.
    - We'll be using the NTC thermistor to measure temperature in our exercise.
<img style="display: block; margin: auto;" alt="pico2" src="./images/ntc-resistor.png"/>

- PTC (Positive Temperature Coefficient): 
    - Resistance increases as temperature rises.
    - They primarily protect against overcurrent and overtemperature conditions as resettable fuses and are commonly used in air conditioners, medical devices, battery chargers, and welding equipment.


## Reference
- [Thermistor Basics](https://www.teamwavelength.com/thermistor-basics/)
- [Thermistors](https://www.electronics-tutorials.ws/io/thermistors.html)
