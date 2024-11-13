## Turn on LED(or Lamp) in low Light with Pico 

In this exercise, we'll control an LED based on ambient light levels. The goal is to automatically turn on the LED in low light conditions. 

You can try this in a closed room by turning the room light on and off. When you turn off the room-light, the LED should turn on, given that the room is dark enough, and turn off again when the room-light is switched back on. Alternatively, you can adjust the sensitivity threshold or cover the light sensor (LDR) with your hand or some object to simulate different light levels.

Note: You may need to adjust the ADC threshold based on your room's lighting conditions and the specific LDR you are using.

# TODO: Explanation

```rust
bind_interrupts!(struct Irqs {
    ADC_IRQ_FIFO => InterruptHandler;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());
    let mut adc = Adc::new(p.ADC, Irqs, Config::default());

    let mut p26 = Channel::new_pin(p.PIN_26, Pull::None);
    let mut led = Output::new(p.PIN_15, Level::Low);

    loop {
        let level = adc.read(&mut p26).await.unwrap();
        if level > 3800 {
            led.set_high();
        } else {
            led.set_low();
        }
        Timer::after_secs(1).await;
    }
}
```
