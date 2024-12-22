## Beeping with an Active Buzzer

Since you already know that an active buzzer is simple to use, you can make it beep just by powering it. In this exercise, we'll make it beep with just a little code.


### Hardware Requirements
- **Active Buzzer**
- **Female-to-Male** or **Male-to-Male** (depending on your setup)

We'll use the Embassy HAL for this project.


## Project from template

To set up the project, run:
```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git
```
When prompted, give your project a name, like "active-beep" and select `embassy` as the HAL.

Then, navigate into the project folder:
```sh
cd PROJECT_NAME
# For example, if you named your project "active-beep":
# cd active-beep
```

All you need to do is change the output pin from 25 to 15 in the template code.

```rust
// Active Buzzer
#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());
    let mut buzzer = Output::new(p.PIN_15, Level::Low); // Changed PIN number to 15

    loop {
        buzzer.set_high();
        Timer::after_millis(500).await;

        buzzer.set_low();
        Timer::after_millis(500).await;
    }
}
```


## Clone the existing project
You can clone (or refer) project I created and navigate to the `active-beep` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/active-beep
```

