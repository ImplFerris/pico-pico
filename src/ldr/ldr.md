## How LDR works?

We have already given an introduction to what an LDR is. Let me repeat it again: an LDR changes its resistance based on the amount of light falling on it. The brighter the light, the lower the resistance, and the dimmer the light, the higher the resistance.

Dracula: Think of the LDR as Dracula. In sunlight, he gets weaker (just like the resistance gets lower). But in the dark, he gets stronger (just like the resistance gets higher).

We will not cover what kind of semiconductor materials are used to make an LDR, nor why it behaves this way in depth. I recommend you read this [article](https://www.elprocus.com/ldr-light-dependent-resistor-circuit-and-working/) and do further research if you are interested.

## Simulation of LDR in Voltage Divider 

I have created a voltage divider circuit with an LDR(a resistor symbol with arrows, kind of indicating light shining on it) in Falstad . You can import the circuit file I created, [`voltage-divider-ldr.circuitjs.txt`](./voltage-divider-ldr.circuitjs.txt), import into the [Falstad site](https://www.falstad.com/circuit/e-voltdivide.html) and play around.

You can adjust the brightness value and observe how the resistance of R2 (which is the LDR) changes. Also, you can watch how the \\( V_{out} \\) voltage changes as you increase or decrease the brightness.

### Example output for full brightness
The resistance of the LDR is low when exposed to full brightness, causing the output voltage(\\( V_{out} \\)) to be significantly lower.

<img style="display: block; margin: auto;" alt="voltage-divider-ldr1" src="./images/voltage-divider-ldr1.png"/>


### Example output for low light
With less light, the resistance of the LDR increases and the output voltage increase.

<img style="display: block; margin: auto;" alt="voltage-divider-ldr2" src="./images/voltage-divider-ldr2.png"/>

### Example output for full darkness
In darkness, the LDR's resistance is high, resulting in a higher output voltage (\\( V_{out} \\)).

<img style="display: block; margin: auto;" alt="voltage-divider-ldr3" src="./images/voltage-divider-ldr3.png"/>

