# Write Data to an SD Card Using Embedded Rust on Raspberry Pi Pico

In this chapter, we create a file on a microSD card and write data into it. The SD card wiring, SPI configuration, logging setup, and card initialization remain unchanged. We only change how the file is opened and how data is written to it.

If the file already exists on the card, it will be overwritten. After the program runs, you can remove the card and verify the contents on your computer.


## Project Setup

You can either reuse the project from the read example and modify it, or create a new project from the template and follow the same setup steps up to creating the sdcard instance.

From that point onward, only the file handling code is different.

## Opening a file for writing

To write data, the file must be opened in a mode that allows creation and modification. Here we use ReadWriteCreateOrTruncate, which creates the file if it does not exist and truncates it if it already exists. This ensures that the file starts empty each time the program runs.

```rust
let my_file = root_dir
    .open_file_in_dir(
        "FERRIS.TXT",
        embedded_sdmmc::Mode::ReadWriteCreateOrTruncate,
    )
    .expect("failed to create FERRIS.TXT file");
```

## Writing data to the file

To keep the example simple, we write a single line of text into the file. If the write succeeds, the file is flushed to ensure that the data is committed to the SD card.


```rust
let line = "Hello, Ferris!";
if let Ok(()) = my_file.write(line.as_bytes()) {
    info!("Written Data");
    if let Err(_) = my_file.flush() {
        info!("Failed to flush");
    }
} else {
    error!("Unable to write the data");
}
```

## Clone the existing project

You can clone (or refer) project I created and navigate to the `write-sdcard` folder.

```sh
git clone https://github.com/ImplFerris/pico2-embassy-projects
cd pico2-embassy-projects/sdcard/write-sdcard/
```

## Verifying the result

After the program finishes running, power off the board and remove the microSD card. When the card is inserted into a computer, a file named FERRIS.TXT should be present in the root directory.

> [!Tip]
> In this example, we are using a dummy time source, so the file timestamp will appear around 1980. If you want correct timestamps, you will need to set up an RTC and provide a custom TimeSource implementation that returns the current time.

Opening the file should show:

```text
Hello, Ferris!
```
