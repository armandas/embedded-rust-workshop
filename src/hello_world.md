# Hello World

## Create the project

Let's create our first embedded Rust project. For this, we will be using `esp-generate` tool installed earlier.

To begin with, run the following command:

```text
esp-generate --headless --chip=esp32c6 -o log -o unstable-hal hello_world
```

This will create a new project in `hello_world` directory. Go ahead and open it with: `code hello_world`.

> 💡 You can run `esp-generate --chip=esp32c6` to open the tool in interactive mode and explore the available options.

## Run

With the ESP board plugged into your computer, you should be able to run the firmware with:

```text
cargo run
```

You may have to select the appropriate device. If this is the case, the tool will prompt for it.

If everything goes well, your device will boot and start printing the "hello world" message:

```text
INFO - Hello world!
INFO - Hello world!
INFO - Hello world!
INFO - Hello world!
INFO - Hello world!
```

## Experiment

Experiment with this example a little bit.

- Change the message to print something else.
- Change the log level (e.g. to `warn`).
- Change the printing frequency by adjusting the sleep duration.

## Bonus

The way the delay is currently written is a bit verbose.
We can simplify the code by using [`Delay`][1] from the `esp_hal` crate.

1. Add the following line next to the existing `use` declarations:
   ```rust
   use esp_hal::delay::Delay;
   ```
2. Create a delay variable:
   ```rust
   let delay = Delay::new();
   ```
3. Replace the two lines using `delay_start` with the following one line:
   ```rust
   delay.delay_millis(1000);
   ```

[1]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0-rc.0/esp32c6/esp_hal/delay/struct.Delay.html
