# Dimming LED

In this section, we will learn how to create a dimming effect (i.e. reducing and increasing the brightness gradually) for an LED using the Raspberry Pi Pico 2. First, we will dim the onboard LED, which is connected to GPIO pin 25 (based on the datasheet).

To make it dim, we use a technique called PWM (Pulse Width Modulation). You can refer to [the intro in the PWM section](../../core-concepts/pwm/index.md).

We will gradually increment the PWM's duty cycle to increase the brightness, then we gradually decrement the PWM duty cycle to  reduce the brightness of the LED. This effectively creates the dimming LED effect.

## The Eye

"
Come in close... Closer...

Because the more you think you see... The easier it’ll be to fool you...

Because, what is seeing? ... You're looking but what you're really doing is filtering, interpreting, searching for meaning...
"

Here's the magic: when this switching happens super quickly, our eyes can't keep up. Instead of seeing the blinking, it just looks like the brightness changes! The longer the LED stays ON, the brighter it seems, and the shorter it's ON, the dimmer it looks. It's like tricking your brain into thinking the LED is smoothly dimming or brightening.


## Core Logic

What we will do in our program is gradually increase the duty cycle from a low value to a high value in the first loop, with a small delay between each change. This creates the fade-in effect. After that, we run another loop that decreases the duty cycle from high to low, again with a small delay. This creates the fade-out effect.

You can use the onboard LED, or if you want to see the dimming more clearly, use an external LED. Just remember to update the PWM slice and channel to match the GPIO pin you are using.
