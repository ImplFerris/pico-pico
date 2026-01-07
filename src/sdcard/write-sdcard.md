# Writing into SD Card with Raspberry Pi Pico

Once you've completed the SD card reading part, this step is straightforward. Simply open the file in ReadWriteCreateOrTruncate mode. If the file doesn't exist, it will be created; if it does exist, it will be truncated. After that, you can write to the file.

We will be creating a file called `FERRIS.TXT` and then write Ferris emoji (🦀) into it.

```rust
let Ok(mut my_file) = root_dir.open_file_in_dir(
    "FERRIS.TXT",
    embedded_sdmmc::Mode::ReadWriteCreateOrTruncate,
) else {
    serial.write("err in open_file_in_dir".as_bytes()).unwrap();
    continue;
};
my_file.write("🦀".as_bytes()).unwrap();
serial.write("Written".as_bytes()).unwrap();
```

To verify, you can either connect the SD card directly to your computer or modify the SD card reading program to read and print the content of this file.
