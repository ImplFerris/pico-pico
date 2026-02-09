# Async In Embedded Rust

When I first started this book, I wrote most of the examples using `rp-hal` only. In this revision, I have rewritten the book to focus mainly on `async` programming with Embassy. The official [Embassy book](https://embassy.dev/book/) already has good documentation, but I want to give a short introduction here. Let's have a brief look at async and understand why it's so valuable in embedded systems.

## Imagine You're Cooking Dinner

If you're familiar with concurrency and async concepts, you don't need this analogy; Embassy is basically like Tokio for embedded systems, providing an async runtime. If you're new to async, let me explain with this analogy.

You are making dinner and you put water on to boil. Instead of standing there watching, you chop vegetables. You glance at the pot occasionally, and when you see bubbles, you're ready for the next step. Now while the main dish cooks, you prepare a side dish in another pan. You even check a text message on your phone. You're one person constantly moving between tasks, checking what needs attention, doing work whenever something is ready, and never just standing idle waiting.

<div class="image-with-caption" style="text-align:center; ">
    <img src="./images/cooking-scenario.jpg" alt="Cooking" style="height:auto; display:block; margin:0 auto;"/>
    <div class="caption" style="font-size:0.9em; color:#555; margin-top:6px;">Photo by <a href="https://unsplash.com/@jsnbrsc?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Jason Briscoe</a> on <a href="https://unsplash.com/photos/man-cutting-vegetables-5IGprlBT5g4?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></div>
</div>

That's async programming. You're the executor, constantly deciding what needs attention. Each cooking task is an async operation. The stove does its heating without you watching it. That's the non-blocking wait. You don't freeze in place staring at boiling water. You go do other productive work and come back when it's ready. The key insight is efficient orchestration: one person (the executor), multiple waiting tasks, and you're always doing something useful by switching your attention to whatever is ready right now. This is exactly what async programming does for your microcontroller.

## Different Approaches

In embedded systems, your microcontroller spends a lot of time waiting. It waits for a button press, for a timer to expire, or for an LED to finish blinking for a set duration. Without async, you have two main approaches.

### Blocking

The first approach is blocking code. Your program literally stops and waits. If you're waiting for a button press, your code sits in a loop checking if the button state has changed. During this time, your microcontroller can't do anything else. It can't blink an LED, it can't check other buttons, it can't respond to timers. All of your processor's power is wasted in a tight loop asking "is it ready yet?" over and over again.

### Interrupt

The second approach is using interrupts directly. When hardware events happen, like a button being pressed or a timer expiring, the interrupt handler runs. This is better because your main code can keep running, but interrupt-based code quickly becomes complex and error-prone. You need to carefully manage shared state between your main code and interrupt handlers.

Do not worry about interrupts for now. We will go into them in more depth in later chapters.

### Async

Async programming gives you the best of both worlds. Your code looks clean and sequential, like blocking code, but it doesn't actually block. When you await something, your code says "I need to wait for this, but feel free to do other work in the meantime." The async runtime, which Embassy provides for us, handles all the complexity of switching between tasks efficiently.

## How Async Works in Rust

When you write an async function in Rust, you use the `async` keyword before `fn`. Inside that function, you can use the `await` keyword on operations that might take time. Here's what it looks like:

```rust
async fn blink_led(mut led: Output<'static>) {
    loop {
        led.set_high();
        Timer::after_millis(500).await;
        led.set_low();
        Timer::after_millis(500).await;
    }
}
```

The important part is the `.await`. When you write `Timer::after_millis(500).await`, you're telling the runtime "I need to wait 500 milliseconds, but I don't need the CPU during that time." The runtime can then go run other tasks. When the 500 milliseconds are up, your task resumes right where it left off.

Think back to our cooking analogy. When you put something on the stove and walk away, you're essentially "awaiting" it to be ready. You do other things, and when it's done, you return to that task. Just like you act as the executor in the kitchen, keeping track of what needs attention and when, the async runtime plays the same role for your program.

## Embassy

Embassy is one of the popular async runtime that makes all of this work in embedded Rust. It provides the executor that manages your tasks, handles hardware interrupts.

### Executor

When you use `#[embassy_executor::main]`, Embassy automatically sets everything up - it runs your tasks, puts the CPU to sleep when everything is waiting, and wakes it up when hardware events occur.  The Executor is the coordinator that decides which task to poll when. The executor maintains a queue of tasks that are ready to run. When a task hits await and yields, the executor moves to the next ready task. When there are no tasks ready to run, the executor puts the CPU to sleep. Interrupts wake the executor back up, which then polls any tasks that became ready.

## RTIC

RTIC (Real-Time Interrupt-driven Concurrency) is another popular framework for embedded Rust. Unlike Embassy, which provides an async runtime along with hardware drivers, RTIC focuses only on execution and scheduling. In RTIC, you declare tasks with fixed priorities and shared resources upfront, and the framework checks at compile time that resources are shared safely without data races. Higher-priority tasks can preempt lower-priority ones, and the scheduling is handled by hardware interrupts, which makes timing very predictable. This makes RTIC a good fit for hard real-time systems where precise control and determinism matter.  You can refer the [official RTIC book](https://rtic.rs/2/book/en/) for more info.

In this book, we will mainly use Embassy.
