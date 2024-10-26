# Ferris on OLED

In this task, we will display the `ferris.bmp`([Download](./assets/ferris.bmp) and put it in the project folder) file on the OLED screen.
 
<img style="display: block; margin: auto;" alt="pico2" src="./assets/ferris.bmp"/>

Follow the same instructions as in the "Hello Rust" program, with just a few modifications.  

First, we need to add the tinybmp crate to handle BMP file loading. Use the following Cargo command to include it in your project:
```sh
cargo add tinybmp@0.6.0
```

## Additional imports
```rust
use embedded_graphics::image::Image;
use tinybmp::Bmp;
```

## Difference

After initializing the display, we will load the ferris.bmp file using the tinybmp crate, and then we will draw the image.
```rust
let bmp = Bmp::from_slice(include_bytes!("../ferris.bmp")).unwrap();
let im = Image::new(&bmp, Point::new(32, 0));
im.draw(&mut display).unwrap();
```

## Clone the existing project
You can clone (or refer) project I created and navigate to the `ferris-oled` folder.

```sh
git clone https://github.com/ImplFerris/pico2-projects
cd pico2-projects/ferris-oled
```
