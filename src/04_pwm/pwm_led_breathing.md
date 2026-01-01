# Exercise: LED breathing effect

We learnt how to control the LED brightness and make the brightness ramp-up.
For this exercise, you have to implement a commonly-used breathing effect.

The sequence is roughly as follows:

1. LED brightness should increase gradually until it reaches the maximum.
1. LED brightness should decrease gradually until it reaches zero.
1. LED stays off for a couple of seconds.
1. The sequence repeats.

<details>
<summary><strong>Solution</strong></summary>

The requirements of this exercise a suited perfectly for a [state machine][1]
and Rust's `enum`s work great for state machines.

Let's define our enum. We can achieve our goals by using two variants (states):
one for increasing brightness and one for decreasing.
The enum variants will hold the actual brightness value.

```rust
enum LedState {
    Up(u16),
    Down(u16),
}
```

Let's create the instance of our `LedState`. Declare this variable just above the main loop:

```rust
let mut led_state = LedState::Up(0);
```

We will start at zero brightness and increasing direction.

Finally, let's implement our state transitions:

```rust
loop {
    match led_state {
        // We have reached maximum, change state to ramp down.
        LedState::Up(255) => {
            led.set_timestamp(255);
            led_state = LedState::Down(255);
        }
        // Ramping up: increment the brightness.
        LedState::Up(brightness) => {
            led.set_timestamp(brightness);
            led_state = LedState::Up(brightness + 1);
        }
        // We have reached minimum, wait two seconds then change state to ramp up.
        LedState::Down(0) => {
            led.set_timestamp(0);
            delay.delay_millis(2000);
            led_state = LedState::Up(0);
        }
        // Ramping down: decrement the brightness.
        LedState::Down(brightness) => {
            led.set_timestamp(brightness);
            led_state = LedState::Down(brightness - 1);
        }
    }
    delay.delay_millis(10);
}
```

</details>

[1]: https://en.wikipedia.org/wiki/Finite-state_machine
