# Buzzinga

In this section, we'll explore some fun activities using the `buzzer`. I chose the title "Buzzinga" just for fun (a nod to Sheldon's "Bazinga" in *The Big Bang Theory*); it's not a technical term.

- **Passive Buzzer**
- **Jumper Wires**:
  - **Female-to-Male** jumper wires for connecting the Pico 2 to the buzzer pins (Positive and Ground).

The buzzer has two pins:  Positive(Signal), Ground; The positive side of the buzzer is typically marked with a **+** symbol and is the longer pin, while the negative side (ground) is the shorter pin, similar to an LED. However, some passive buzzers may allow for either pin to be connected to ground or signal, depending on the specific model. 

By the way, I used an active buzzer in my experiment. A passive buzzer is recommended if you plan to play different sounds, as it provides a better tone.

## Connection Overview
<table style="margin-bottom:20px">
  <thead>
    <tr>
      <th>Pico Pin</th>
      <th style="width: 250px; margin: 0 auto;">Wire</th>
      <th>Buzzer Pin</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>GPIO 15</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire red" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Positive Pin</td>
      <td>Receives PWM signals to produce sound.</td>
    </tr>
    <tr>
      <td>GND</td>
      <td style="text-align: center; vertical-align: middle; padding: 0;">
        <div class="wire black" style="width: 200px; margin: 0 auto;">
          <div class="male-left"></div>
          <div class="male-right"></div>
        </div>
      </td>
      <td>Ground Pin</td>
      <td>Connects to ground.</td>
    </tr>
  </tbody>
</table>


<img style="display: block; margin: auto;" alt="pico2" src="./images/pico-buzzer-circuit.png"/>

Before moving forward, make sure you've read the following sections and understood the concepts.
- [PWM introduction](../core-concepts/pwm/index.md) in the Blink LED section
- [More on PWM](../servo/pwm.md) in the servo section
- [Calculating top](../servo/servo-pico.md) in the servo section


## Reference
- [Pico official guide on buzzer](https://projects.raspberrypi.org/en/projects/introduction-to-the-pico/9)
