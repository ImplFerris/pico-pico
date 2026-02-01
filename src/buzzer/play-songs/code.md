## Playing the Game of Thrones Melody

In this section, we put everything together and work in the main.rs file.

By this point, we already have the note frequencies, song timing logic, and melody data. Here, we just wire them together using PWM and timers.

## Imports

Add the required imports for PWM, timers, and song handling.

```rust
// For PWM
use embassy_rp::pwm::{Config as PwmConfig, Pwm, SetDutyCycle};

use crate::music::Song;
```

## PWM and Buzzer

Unlike the previous example, we clone PwmConfig. The Pwm constructor consumes the config object, but inside the loop we need to modify top to change the PWM frequency for each musical note.

```rust
let mut pwm_config = PwmConfig::default();
pwm_config.top = PWM_TOP;
pwm_config.divider = PWM_DIV_INT.into();

let mut buzzer = Pwm::new_output_b(p.PWM_SLICE7, p.PIN_15, pwm_config.clone());
```

## Create the Song object

Create a Song using the tempo defined for the Game of Thrones theme.

```rust
let song = Song::new(got::TEMPO);
```

## Playing the notes

The melody is played by looping through the MELODY array. Each entry contains a note frequency and a duration value.

```rust
// One time play the song
for (note, duration_type) in got::MELODY {
    let top = get_top(note, PWM_DIV_INT);
    pwm_config.top = top;
    buzzer.set_config(&pwm_config);

    let note_duration = song.calc_note_duration(duration_type);
    let pause_duration = note_duration / 10; // 10% of note_duration

    buzzer
        .set_duty_cycle_percent(50)
        .expect("50 is valid duty percentage"); // Set duty cycle to 50% to play the note

    Timer::after_millis(note_duration - pause_duration).await; // Play 90%

    buzzer
        .set_duty_cycle_percent(0)
        .expect("50 is valid duty percentage"); // Stop tone
    Timer::after_millis(pause_duration).await; // Pause for 10%
}
```
For each note, the PWM frequency is updated by setting a new top value. This makes the buzzer produce the correct pitch.

The note duration is calculated from the song tempo. Most of that time is spent playing the note, and a small part is left silent. That short silence helps separate notes so the melody sounds cleaner.

The buzzer is played by setting the duty cycle to 50 percent and stopped by setting it to zero.


### Keeping the Program Running

After the melody finishes, this is just to keep the program alive.

```rust
loop {
    Timer::after_millis(100).await;
}
```

## Clone the existing project

You can clone (or refer to) the project I created and navigate to the `buzzer-song` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/buzzer-song
```

## rp-hal version
 
If you want to see the same example implemented using rp-hal, you can find it here.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-rp-projects/got-buzzer
```

