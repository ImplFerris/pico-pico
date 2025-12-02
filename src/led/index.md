# Dimming LED

In this section, we will learn how to create a dimming effect(i.e. reducing and increasing the brightness gradually) for an LED using the Raspberry Pi Pico 2. First, we will dim the onboard LED, which is connected to GPIO pin 25 (based on the datasheet). 

To make it dim, we use a technique called PWM (Pulse Width Modulation). You can refer to the intro to the PWM section [here](../core-concepts/pwm/index.md).

We will gradually increment the PWM's duty cycle to increase the brightness, then we gradually decrement the PWM duty cycle to  reduce the brightness of the LED. This effectively creates the dimming LED effect. 


## The Eye

"
Come in close... Closer... 

Because the more you think you see... The easier it’ll be to fool you... 

Because, what is seeing?.... You're looking but what you're really doing is filtering, interpreting, searching for meaning...
"

Here's the magic: when this switching happens super quickly, our eyes can't keep up. Instead of seeing the blinking, it just looks like the brightness changes! The longer the LED stays ON, the brighter it seems, and the shorter it's ON, the dimmer it looks. It's like tricking your brain into thinking the LED is smoothly dimming or brightening.

## Fading Up

The code below gradually increases the LED brightness by changing the duty cycle from 0 to 25,000, with a small delay between each step:

```rust
for i in LOW..=HIGH {
  delay.delay_us(8);
  let _ = channel.set_duty_cycle(i);
}
```

The delay ensures the LED brightens gradually.  We need the delay between them; otherwise, it would quickly jump from dim to bright. The delay allows for a smooth, noticeable "fading up" effect.

Dont' believe me! Adjust the delay to 0 and observe. You can increase the delay (eg: 25) and observe the fading effect.

> **Note:** The `set_duty_cycle` function writes the given value into the CC register (Compare Counter), which determines when the PWM output switches from HIGH to LOW.


## Fading Down

The following code reduce the LED brightness by decrementing the duty cycle from 25,000 to 0.
```rust
// Here rev is to reverse the iteration. so it goes from 25_000 to 0
for i in (LOW..=HIGH).rev() {  
    delay.delay_us(8);
    let _ = channel.set_duty_cycle(i);
}
```

## Pause

After fading up and down, the program pauses for 500 milliseconds before repeating the cycle, allowing the LED to rest briefly.

Play around by adjusting the `delay` and observe. You can even comment out one of the for loop and observe the effect. 

