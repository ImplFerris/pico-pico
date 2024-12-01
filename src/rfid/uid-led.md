## Turn on LED on UID Match
In this section, we'll use the UID obtained in the previous chapter and hardcode it into our program. The LED will turn on only when the matching RFID tag is nearby; otherwise, it will remain off. When you bring the RFID tag close, the LED will light up. If you bring a different tag, like a key fob or any other RFID tag, the LED will turn off.

## Logic
It is very simple straightforward logic.

```rust
let mut led = pins.gpio25.into_push_pull_output();

// Replace the UID Bytes with your tag UID
const TAG_UID: [u8; 4] = [0x13, 0x37, 0x73, 0x31];

loop {
    led.set_low().unwrap();

    if let Ok(atqa) = rfid.reqa() {
        if let Ok(uid) = rfid.select(&atqa) {
            if *uid.as_bytes() == TAG_UID {
                led.set_high().unwrap();
                timer.delay_ms(500);
            }
        }
    }
}

```


## Clone the existing project
You can clone (or refer) project I created and navigate to the `rfid-led` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/rfid-led/
```


### Light it Up
Lets flash the pico with our program.
```sh
cargo run
``` 
Now bring the RFID tag near the RFID reader, the onboard LED on the Pico should turn on. Next, try bringing the key fob closer to the reader, and the LED will turn off. Alternatively, you can first read the key fob UID and hardcode it into the program to see the opposite behavior.
