# Melody Example: Game of Thrones Theme

This section contains code snippets for the Rust module got.

## Importing music definitions

The got module uses note constants and helper types defined in the music module. We bring them into scope using the following import:

```rust
use crate::music::*;
```

This allows the melody to use note constants like `NOTE_E4` and `NOTE_A4` directly, without writing the module name each time.

## Tempo

We declare the tempo for the song. You can change this value and observe how it affects playback speed.

```rust
pub const TEMPO: u16 = 85;
```

## Melody Array

We define the melody of the Game of Thrones theme as an array of notes and durations. Each entry is a tuple containing a note frequency and its duration.

The duration is represented by an integer. Positive values represent normal notes. Negative values represent dotted notes.

```rust
pub const MELODY: [(f64, i16); 92] = [
    // Game of Thrones Theme
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_E4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_E4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_E4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 8),
    (NOTE_C4, 8),
    (NOTE_E4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, -4),
    (NOTE_C4, -4),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 4),
    (NOTE_C4, 4),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_D4, -1),
    (NOTE_F4, -4),
    (NOTE_AS3, -4),
    (NOTE_DS4, 16),
    (NOTE_D4, 16),
    (NOTE_F4, 4),
    (NOTE_AS3, -4),
    (NOTE_DS4, 16),
    (NOTE_D4, 16),
    (NOTE_C4, -1),
    // Repeat
    (NOTE_G4, -4),
    (NOTE_C4, -4),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 4),
    (NOTE_C4, 4),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_D4, -1),
    (NOTE_F4, -4),
    (NOTE_AS3, -4),
    (NOTE_DS4, 16),
    (NOTE_D4, 16),
    (NOTE_F4, 4),
    (NOTE_AS3, -4),
    (NOTE_DS4, 16),
    (NOTE_D4, 16),
    (NOTE_C4, -1),
    (NOTE_G4, -4),
    (NOTE_C4, -4),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_G4, 4),
    (NOTE_C4, 4),
    (NOTE_DS4, 16),
    (NOTE_F4, 16),
    (NOTE_D4, -2),
    (NOTE_F4, -4),
    (NOTE_AS3, -4),
    (NOTE_D4, -8),
    (NOTE_DS4, -8),
    (NOTE_D4, -8),
    (NOTE_AS3, -8),
    (NOTE_C4, -1),
    (NOTE_C5, -2),
    (NOTE_AS4, -2),
    (NOTE_C4, -2),
    (NOTE_G4, -2),
    (NOTE_DS4, -2),
    (NOTE_DS4, -4),
    (NOTE_F4, -4),
    (NOTE_G4, -1),
];
```
