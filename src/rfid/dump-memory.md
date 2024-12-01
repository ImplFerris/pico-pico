## Dump Entire Memory

You've learned how to read the data from each block of the first sector(sector 0) by authenticating into it. Now, we will loop through each sector. Re-Authentication is required every time we move to a new sector. For each sector, we will display the 16-byte data from every 4 blocks.

To make it clearer, we'll add some formatting and labels, indicating which sector and block we're referring to (both absolute and relative block numbers to the sector), as well as whether the block is a sector trailer or a data block.


### Loop through the sector

We will create a separate function to loop through all 16 sectors (sectors 0 to 15), read all the blocks within each sector, and print their data.
 
```rust
fn dump_memory<E, COMM: mfrc522::comm::Interface<Error = E>, B: UsbBus>(
    uid: &mfrc522::Uid,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
    serial: &mut SerialPort<B>,
) -> Result<(), &'static str> {
    let mut buff: String<64> = String::new();
    for sector in 0..16 {
        // Printing the Sector number
        write!(buff, "\r\n-----------SECTOR {}-----------\r\n", sector).unwrap();
        serial.write(buff.as_bytes()).unwrap();
        buff.clear();

        read_sector(uid, sector, rfid, serial)?;
    }
    Ok(())
}
```

### Labels

The `read_sector` function follows the same logic as before, but with added formatting and labels. It now prints the absolute block number, the block number relative to the sector, and labels for the manufacturer data (MFD) block and sector trailer blocks.

```rust
fn read_sector<E, COMM: mfrc522::comm::Interface<Error = E>, B: UsbBus>(
    uid: &mfrc522::Uid,
    sector: u8,
    rfid: &mut Mfrc522<COMM, mfrc522::Initialized>,
    serial: &mut SerialPort<B>,
) -> Result<(), &'static str> {
    const AUTH_KEY: [u8; 6] = [0xFF; 6];

    let mut buff: String<64> = String::new();

    let block_offset = sector * 4;
    rfid.mf_authenticate(uid, block_offset, &AUTH_KEY)
        .map_err(|_| "Auth failed")?;

    for abs_block in block_offset..block_offset + 4 {
        let rel_block = abs_block - block_offset;
        let data = rfid.mf_read(abs_block).map_err(|_| "Read failed")?;

        // Prining the Block absolute and relative numbers
        write!(buff, "\r\nBLOCK {} (REL: {}) | ", abs_block, rel_block).unwrap();
        serial.write(buff.as_bytes()).unwrap();
        buff.clear();

        // Printing the block data
        print_hex_to_serial(&data, serial);

        // Printing block type
        let block_type = get_block_type(sector, rel_block);
        write!(buff, "| {} ", block_type).unwrap();
        serial.write(buff.as_bytes()).unwrap();
        buff.clear();
    }
    serial
        .write("\r\n".as_bytes())
        .map_err(|_| "Write failed")?;
    Ok(())
}
```

We will create a small helper function to determine the block type based on the sector and its relative block number.
```rust
fn get_block_type(sector: u8, rel_block: u8) -> &'static str {
    match rel_block {
        0 if sector == 0 => "MFD",
        3 => "TRAILER",
        _ => "DATA",
    }
}
```

### The main loop 
There isn't much change in the main loop. We just call the `dump_memory` function instead of `read_sector`.

```rust
loop {
    let _ = usb_dev.poll(&mut [&mut serial]);
    if let Ok(atqa) = rfid.reqa() {
        if let Ok(uid) = rfid.select(&atqa) {
            if let Err(e) = dump_memory(&uid, &mut rfid, &mut serial) {
                serial.write(e.as_bytes()).unwrap();
            }
            rfid.hlta().unwrap();
            rfid.stop_crypto1().unwrap();
        }
    }
}
```


## Clone the existing project
You can clone (or refer) project I created and navigate to the `rfid-dump` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/rfid-dump/
```

## Dump
When you run the program and bring your tag or key fob close, you should see output like this. If you notice the 0x40..0x43 bytes in the block 18 (the 3rd block of the 4th sector) and wonder why it's there; good catch! That's the custom data I wrote to the tag.

<img style="display: block; margin: auto;" src="./images/rfid-dump.png"/>
