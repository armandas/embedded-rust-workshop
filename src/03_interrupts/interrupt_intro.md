# Interrupts

In the last exercise, we learnt how to read a GPIO pin state.
However, it's not always desirable to poll the pin state in software.
This is where interrupts come it. We can ask hardware to notify us whenever the pin state changes.

## Background

Because interrupts can happen at any time during the execution of our program, we can think of them
as if they're running on a separate thread. As such, when working with interrupts, we apply the same
principles as when working in a multi-threaded environment.

Additionally, because the interrupt routine is called by hardware, there is no way to pass parameters or
return values from the interrupt handler.

One common way to pass data between the application and interrupt contexts is using global state.
This is mostly straightforward in C, as you can just read and write any global variable at any time.
Whether you like it or not, this "feature" opens up questions about what happens when the variable is modified
while some other code is reading it?

Consider this code:

```c
volatile int counter = 0;

int main() {
    while (true) {
        app_task();
        ++counter; // Same as: counter = counter + 1
    }
}

void timer_interrupt() {
    interrupt_task(counter);
    counter = 0;
}
```

In Rust, however, using a mutable global state is explicitly `unsafe`. While it's possible to bypass
the compiler rules, the language strongly encourages developers to write correct code.

## Atomics & Mutexes

When working with primitive types we can use one of the types provided in [`core::sync::atomic`][1].
Atomic types are the most fundamental way to ensure exclusive access -- these opperations are provided by the CPU.

For more complex types, we have to resort to higher level synchronization primitives, such as [Mutexes][2] and
other types of locks. Note that `Mutex` is only available in the `std` library, as is requires operating system support.

No worries though, for embedded systems we can use Mutex provided by the [`critical_section`][3] crate.

[1]: https://doc.rust-lang.org/nightly/std/sync/atomic/index.html
[2]: https://doc.rust-lang.org/nightly/std/sync/struct.Mutex.html
[3]: https://docs.rs/critical-section/latest/critical_section/
