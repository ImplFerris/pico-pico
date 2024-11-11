## Playing the Game of Thrones Melody

This section demonstrates how to play the Game of Thrones melody using PWM (Pulse Width Modulation) for generating the tones and timing the note durations. The code calculates the duration of each note, sets the PWM duty cycle, and controls the timing to ensure proper pauses between notes.

### Song object
This creates a new Song object using the tempo value from got::TEMPO, which is set to 85 BPM.  The Song object will manage the note durations based on the tempo.
```rust
let song = Song::new(got::TEMPO);
```

## Playing the notes
This loop iterates over the muisc-notes array. Each item in the melody is a tuple containing a note and its duration type (e.g., quarter note, eighth note).

```rust
for (note, duration_type) in got::MELODY {
        let top = get_top(note, PWM_DIV_INT);
        pwm.set_top(top);

        let note_duration = song.calc_note_duration(duration_type);
        let pause_duration = note_duration / 10; // 10% of note_duration

        pwm.channel_b.set_duty_cycle_percent(50).unwrap(); // Set duty cycle to 50% to play the note

        timer.delay_ms(note_duration - pause_duration); // Play 90%
        pwm.channel_b.set_duty_cycle(0).unwrap(); // Stop tone
        timer.delay_ms(pause_duration); // Pause for 10%
    }

```

First, we calculate the top value based on the note frequency. This sets the PWM frequency to match the target note.

Next, the `calc_note_duration` function is used to determine how long each note should be played. We also calculate the pause duration as 10% of the note duration. The 90% duration ensures that the note is played clearly, while the 10% pause creates a small gap between notes, resulting in a cleaner and more distinct melody.


### Keeping the Program Running

This loop keeps the program running indefinitely, as required by the main function's signature. The main function has a -> ! return type, meaning it doesn't return.

```rust
loop {
    // Keep the program running
    timer.delay_ms(500);
}
```

## Clone the existing project
You can clone (or refer) project I created and navigate to the `got-buzzer` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/got-buzzer
```

## Wokwi Project
An Arduino version of this exercise is available on the Wokwi site with the Pico board. Unfortunately, the site currently does not support coding in Rust for the Pico. However, you can refer to this project to understand how it works.

[Project Link ](https://wokwi.com/projects/414268602809291777)
