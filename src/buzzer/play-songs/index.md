# Playing Songs on a Passive Buzzer Using Rust and Raspberry Pi Pico

In this section, we will play songs on a buzzer using the Raspberry Pi Pico.  

If you are not familiar with musical notes or sheet music, you can check the basic theory explained [here](./music-theory.md). This part is optional and only meant to give enough background to follow the example.

For clarity, the code is split into Rust modules. You can also keep everything in a single file, as we have done so far, but splitting it makes the example easier to follow:

- [`music`](./music-module.md)
- [`got`](./song-module.md)

A passive buzzer is recommended for this exercise, though you can use either a passive or an active buzzer.

## PWM

We will use PWM to control the frequency of the signal sent to the buzzer. Each frequency corresponds to a musical note. The frequency (musical note) is held for a specific duration before switching to the next note, based on the music data.

For example, the note `A4` is 440 Hz. To play this note, we configure the PWM output to 440 Hz and keep it active for the required duration before moving to the next note.

If you are not familiar with PWM on the Pico, I recommend reading the [PWM section](../../core-concepts/pwm/index.md) before continuing.

## Song Repository

In this exercise, we will play a theme on the buzzer as a demonstration.  

You can also refer to the `rust-embedded-songs` repository and try other songs:

https://github.com/ImplFerris/rust-embedded-songs/

## Submodules

Update `main.rs` to define the submodules, then create the corresponding source files.

```rust
pub mod music;
pub mod got;
```
