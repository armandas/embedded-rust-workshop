# Exercise: toggle LED with interrupt

Now that we have set up the GPIO interrupt, try using it to toggle the LED.
There are many ways to achieve this, so let's set some specific requirements:

1. Use [`AtomicBool`][1]

   We have already seen how to use a Mutex, so for this exercise, let's try something new.

1. Toggle the LED in the main loop

   In general, we want to avoid doing any work in the interrupt handler,
   so use the `AtomicBool` to signal to the main loop that the LED should be toggled.
   Remove any other code from the loop for now.

1. Use `FallingEdge` interrupt type
1. Take care of debouncing

   You will encounter switch bounce - several interrupt events occurring from a single button press.
   Try to come up with a solution for this problem so that the LED is only toggled once per button press.
   There are a multitude of ways to implement this so any solution that works is fine.

<details>
<summary><strong>Solution</strong></summary>

First, we'll need to import `atomic` and then declare our static object object.

```rust
use core::sync::atomic;

static BUTTON_EVENT: atomic::AtomicBool = atomic::AtomicBool::new(false);
```

Next, set the `BUTTON_EVENT` in the `button_handler()`:

```rust
BUTTON_EVENT.store(true, atomic::Ordering::Relaxed);
```

Finally, in the main loop, let's check the `BUTTON_EVENT` and if it's `true`, toggle the LED.

```rust
loop {
    if BUTTON_EVENT.load(atomic::Ordering::Relaxed) {
        led.toggle();
        // Delay before resetting the BUTTON_EVENT acts as debouncing logic.
        delay.delay_millis(150);
        BUTTON_EVENT.store(false, atomic::Ordering::Relaxed);
    }
}
```

</details>

[1]: https://doc.rust-lang.org/nightly/std/sync/atomic/struct.AtomicBool.html
