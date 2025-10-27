# PWM

In this chapter, we will learn how to digitally control the brightness of the LED using [Pulse-width modulation][1].
We will continue using the same `hello_world` project from the previous workshops.

`esp-hal` crate provides a PWM driver in [`mcpwm`][2] module. Let's start by importing the necessary modules and types:

```rust
use esp_hal::mcpwm::operator::PwmPinConfig;
use esp_hal::mcpwm::timer::PwmWorkingMode;
use esp_hal::mcpwm::{McPwm, PeripheralClockConfig};
use esp_hal::time::Rate;
```

Next, let set up the PWM peripheral:

```rust
let clock_cfg = PeripheralClockConfig::with_frequency(Rate::from_mhz(40)).unwrap();
let mut mcpwm = McPwm::new(peripherals.MCPWM0, clock_cfg);
let timer_clock_cfg = clock_cfg
    .timer_clock_with_frequency(255, PwmWorkingMode::Increase, Rate::from_khz(20))
    .expect("could not determine parameters for the requested frequency");

mcpwm.operator0.set_timer(&mcpwm.timer0);
mcpwm.timer0.start(timer_clock_cfg);
```

> 💡 Note the use of `expect`. `expect` performs the same function as `unwrap`, but allows us to write a helpful
> error message. In general, use of `unwrap` is discouraged outside of "example" code. Ideally, the errors should be
> handled gracefully or at least, provide an explanation of what has happened.

We'll change the LED pin to GPIO16, so we can have active-high configuration:

```rust
let led = Output::new(
    peripherals.GPIO16,
    esp_hal::gpio::Level::Low,
    OutputConfig::default(),
);
```

Then, we will transfer GPIO to the PWM instance:

```rust
let mut led = mcpwm.operator0.with_pin_a(led, PwmPinConfig::UP_ACTIVE_HIGH);
```

> 💡 Note how we're re-using the name `led`. This is called rebinding and allows us to write cleaner code.

Finally, let's modify our application to cycle through different brightness levels in a loop:

```rust
let mut sequence = [32, 128, 255].iter().cycle();
loop {
    led.set_timestamp(*sequence.next().unwrap());
    delay.delay_millis(1000);
}
```

[1]: https://en.wikipedia.org/wiki/Pulse-width_modulation
[2]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0-rc.0/esp32c6/esp_hal/mcpwm/index.html
