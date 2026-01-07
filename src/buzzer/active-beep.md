## Beeping with an Active Buzzer

Since you already know how an active buzzer works, we can make it beep by simply turning a GPIO pin on and off.  In this exercise, we use a GPIO pin to power the buzzer, wait for a short time, turn it off, and repeat. This creates a clear beeping sound.

> [!Note]
> This example is meant for an active buzzer. If you use a passive buzzer instead, the sound may be strange or inconsistent. Try this exercise only with an active buzzer.

### Hardware Requirements
- Active buzzer
- Jumper wires (female-to-male or male-to-male, depending on your setup)

## Project from template

Create a new project:

To set up the project, run:
```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.1
```
When prompted, give your project a name, like "active-beep" and select `embassy` as the HAL.

## Main logic

Ensure the buzzer is connected to GPIO 15. The pin is toggled every 500 milliseconds to turn the buzzer on and off.

```rust
let mut buzzer = Output::new(p.PIN_15, Level::Low);

loop {
    buzzer.set_high();
    Timer::after_millis(500).await;

    buzzer.set_low();
    Timer::after_millis(500).await;
}
```


## Clone the existing project
You can clone (or refer) project I created and navigate to the `active-beep` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/active-beep
```

