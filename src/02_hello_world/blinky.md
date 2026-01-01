# Blinky

We have created a typical "hello world" application, but in embedded systems,
the equivalent of "hello world" is blinky: simply blinking an LED.

We will continue working in the `hello_world` project.

## Change the program

In order to blink an LED, we need to define an output pin and make a couple of other changes.

Firstly, let's rename the `_peripherals` variable to `peripherals`.

> 💡 In Rust, a variable can be prefixed with an underscore (`_`) to indicate that it is unused.
> Since we'll be using it, let's remove the leading underscore.

Next, we will define our LED:

```rust
let mut led = Output::new(
    peripherals.GPIO19,
    Level::Low,
    OutputConfig::default().with_drive_mode(DriveMode::OpenDrain),
);
```

The code above will not compile as is, since the compiler won't know where `Output` and other types are coming from. To fix that, import the required types like so:

```rust
use esp_hal::gpio::{DriveMode, Level, Output, OutputConfig};
```

> 💡 Whenever a you have a missing declaration error, you can use `rust-analyzer` to help you fix it.
>
> Place the cursor on the missing type name and press `Ctrl` + `.` on the keyboard.
> In the context menu, select `Import <Type>` or `Qualify <Type>`.
>
> "Import" will add the required `use` declaration, while "Qualify" will prefix the module path in-place.

Finally, add the following line to the body of the `loop` to toggle the LED every cycle:

```rust
led.toggle();
```

## Experiment

`Output` has other methods than `toggle`. Try using `set_high` and `set_low` methods to change the LED blinking pattern.
