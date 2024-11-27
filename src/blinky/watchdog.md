# Watchdog

In January 1994, the Clementine spacecraft successfully mapped the moon. While traveling toward the asteroid Geographos, a floating point exception occurred on May 7, 1994, in the Honeywell 1750 processor, which was responsible for telemetry and various spacecraft functions.


<img style="display: block; margin: auto;" width="400" alt="pico2" src="../images/homer-panic.jpg"/>

The 1750 had a built-in watchdog timer but it was not utilized. The software team later regretted this decision and noted that a standard watchdog might not have been robust enough to detect the failure mode.

So, What exactly is a watchdog? 

You might have already figured out its purpose.

## What is watchdog?
A watchdog timer (WDT) is a hardware component used in embedded systems, its primary purpose is to detect software anomalies and automatically reset the processor if a malfunction occurs, ensuring that the system can recover without human intervention.

## How It Works?

The watchdog timer functions like a counter that counts down from a preset value to zero. The embedded software is responsible for periodically "feeding the dog" (also known as "kicking the dog," a term I don't like) by resetting the counter before it reaches zero. If the software fails to reset the counter (perhaps due to an infinite loop or a system hang), the watchdog timer assumes there's a problem and triggers a reset of the processor. This allows the system to restart and return to normal operation.

**Feeding the dog:** 

Think of a watchdog timer like a dog that needs regular feeding to stay healthy and active. Just as you must feed your dog at scheduled intervals, the watchdog timer requires periodic resets to ensure that the embedded system is operating correctly. Imagine the dog's energy levels decreasing over time. If it runs out of energy, it will bark to alert you (just like the watchdog timer triggers an alert if it reaches zero). To keep the dog happy and active, you need to feed it regularly (or reset the timer) before it runs out of energy!

<img style="display: block; margin: auto;" width="400" alt="pico2" src="../images/watchdog.jpg"/>

By implementing a watchdog timer, embedded systems can be made self-reliant, essential for devices that may be unreachable by human operators, such as space probes or other remote applications.

## Code

In this code snippet, we were setting up the watchdog driver, which is essential for the clock setup process.

```rust
// Set up the watchdog driver - needed by the clock setup code
let mut watchdog = hal::Watchdog::new(pac.WATCHDOG);
```

## References
- [Great Watchdog Timers For Embedded Systems, by Jack Ganssle](https://www.ganssle.com/watchdogs.pdf)
- [Born to fail](https://www.embedded.com/born-to-fail/)
- [A Guide to Watchdog Timers for Embedded Systems](https://interrupt.memfault.com/blog/firmware-watchdog-best-practices)
- [Proper Watchdog Timer Use](https://betterembsw.blogspot.com/2014/05/proper-watchdog-timer-use.html)






