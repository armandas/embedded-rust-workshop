# Delay

The way the delay is currently written is a bit verbose.
We can simplify the code by using [`Delay`][1] from the `esp_hal` crate.

Add the following line next to the existing `use` declarations:

```rust
use esp_hal::delay::Delay;
```
Create a delay variable:

```rust
let delay = Delay::new();
```
Replace the two lines using `delay_start` with the following one line:

```rust
delay.delay_millis(1000);
```

[1]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0-rc.0/esp32c6/esp_hal/delay/struct.Delay.html
