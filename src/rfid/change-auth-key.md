## Changing the Authentication Key

Let's change the authentication key (KeyA) for sector 1. By default, it is set to `FF FF FF FF FF FF`. We'll update it to `52 75 73 74 65 64` which is hex for "Rusted." To do this, we need to modify the trailer block (block 3) of sector 1 while leaving the rest of the sector untouched.

Before proceeding, it is a good idea to verify the current contents of this block. Run the [Dump Memory](./dump-memory.md) or [Read Data](./read-data.md) program to check.

<div class="alert-box alert-box-info">
    <span class="icon"><i class="fa-solid fa-info"></i></span>
    <div class="alert-content">
        <b class="alert-title">Default Keys</b>
        <p>The MIFARE Classic 1K card is pre-configured with the default key FF FF FF FF FF FF for both KeyA and KeyB.  When reading the trailer block, KeyA values are returned as all zeros (00 00 00 00 00 00), while KeyB returned as it is.</p>
    </div>
</div>

We’ll also modify the KeyB contents to verify that the write was successful. We'll set KeyB to the hex bytes of "Ferris" (46 65 72 72 69 73).

Before writing, the access bytes and KeyB values in your block should mostly match what I have, but double-checking is always better than guessing.

Here's the plan:  
1. In the program, we hardcode the default key (`FF FF FF FF FF FF`) into a variable named `current_key`.  
2. Set the `new_key` to `Rusted` (in hex bytes). This is necessary to print the block content after writing; otherwise, we'll get an auth error.
3. The program will print the block's contents both before and after writing.

Once the key is updated, bring the tag nearby again. You will likely see an "Auth failed" error. If you're wondering why, congrats-you figured it out! The new key was successfully written, so the hardcoded `current_key` no longer works. To verify, modify the `read-data` program to use the new key (`Rusted`) and try again.

### Key and Data
The DATA array contains the new KeyA ("Rusted" in hex), access bits, and KeyB ("Ferris" in hex). The current_key is set to the default FF FF FF FF FF FF, and new_key is the first 6 bytes of DATA, which is "Rusted".
 
```rust
let target_sector = 1;
let rel_block = 3;
const DATA: [u8; 16] = [
    0x52, 0x75, 0x73, 0x74, 0x65, 0x64, // Key A: "Rusted"
    0xFF, 0x07, 0x80, 0x69, // Access bits and trailer byte
    0x46, 0x65, 0x72, 0x72, 0x69, 0x73, // Key B: "Ferris"
];
let current_key = &[0xFF; 6];
let new_key: &[u8; 6] = &DATA[..6].try_into().unwrap();
```

### Write Block function
We have slighly modified the write_block function to accept key as argument. 

```rust
fn write_block<E, COMM: mfrc522::comm::Interface<Error = E>>(
    uid: &mfrc522::Uid,
    sector: u8,
    rel_block: u8,
    data: [u8; 16],
    key: &[u8; 6],
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
) -> Result<(), &'static str> {

    let block_offset = sector * 4;
    let abs_block = block_offset + rel_block;

    rfid.mf_authenticate(uid, block_offset, key)
        .map_err(|_| "Auth failed")?;

    rfid.mf_write(abs_block, data).map_err(|_| "Write failed")?;

    Ok(())
}
```

### Read Sector function
We have done similar modification for the read_sector function also.

```rust
fn read_sector<E, COMM: mfrc522::comm::Interface<Error = E>, B: UsbBus>(
    uid: &mfrc522::Uid,
    sector: u8,
    key: &[u8; 6],
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
    serial: &mut SerialPort<B>,
) -> Result<(), &'static str> {
    let block_offset = sector * 4;
    rfid.mf_authenticate(uid, block_offset, key)
        .map_err(|_| "Auth failed")?;

    for abs_block in block_offset..block_offset + 4 {
        let data = rfid.mf_read(abs_block).map_err(|_| "Read failed")?;
        print_hex_to_serial(&data, serial);
        serial
            .write("\r\n".as_bytes())
            .map_err(|_| "Write failed")?;
    }
    Ok(())
}
```

### The main loop
There's nothing new in the main loop. All the read and write functions are ones you've already seen. We're just printing the sector content before and after changing the key.
 
```rust
    loop {
        let _ = usb_dev.poll(&mut [&mut serial]);

        if let Ok(atqa) = rfid.reqa() {
            if let Ok(uid) = rfid.select(&atqa) {
                serial
                    .write("\r\n----Before Write----\r\n".as_bytes())
                    .unwrap();
                if let Err(e) = read_sector(&uid, target_sector, current_key, &mut rfid, &mut serial) {
                    serial.write(e.as_bytes()).unwrap();
                }

                if let Err(e) =
                    write_block(&uid, target_sector, rel_block, DATA, current_key, &mut rfid)
                {
                    serial.write(e.as_bytes()).unwrap();
                }

                serial
                    .write("\r\n----After Write----\r\n".as_bytes())
                    .unwrap();
                if let Err(e) = read_sector(&uid, target_sector, new_key, &mut rfid, &mut serial) {
                    serial.write(e.as_bytes()).unwrap();
                }
                rfid.hlta().unwrap();
                rfid.stop_crypto1().unwrap();
            }
        }
    }
```


## Clone the existing project
You can clone (or refer) project I created and navigate to the `rfid-change-key` folder.

```sh
git clone https://github.com/ImplFerris/pico2-rp-projects
cd pico2-projects/rfid-change-key/
```

## Output
As you can see in the output, when you run the program, it will display the contents of the target block before and after writing. After we change the key, bringing the tag back to the reader will result in an "auth failed" message because the current_key has been changed; The new key is 52 75 73 74 65 64 (Rusted).
 
<img style="display: block; margin: auto;" src="./images/change-auth-key.png"/>

You can also modify the read data program we used earlier with the new key to verify it.
