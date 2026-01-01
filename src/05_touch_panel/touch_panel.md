# Touch panel

In this workshop we'll learn how to set up and use the capacitive touch sensor on our board.

The board manufacturer tells us that the capacitive panel driver IC is AXS5106L.
Typically, in embedded systems development, we would download the part datasheet and start writing a driver for it.
Unfortunately, this chip is one of the obscure Chinese parts and has very little information on it.

Fortunately though, the board manufacturer provides a sample C code and someone already ported that to Rust.
The library we'll be using today is [axs5106l](https://github.com/toto04/axs5106l).

Let's go ahead and add it to our project:

```sh
cargo add axs5106l
```

## Configuration

Hopefully by now you know how to find the documentation and follow the examples, so in this workshop you'll
write most of the code yourself.

Refer to the [`esp_hal`][1] documentation and the [Hardware information][2] page to configure I2C master
peripheral with a 50 kHz clock.

<details>
<summary><strong>Solution</strong></summary>

First, let's import `I2c`. In the example below, we choose not to import `Config` to make it easier to understand
which "Config" we're referring to.

```rust
use esp_hal::i2c::master::I2c;
```

Then, configure the I2C peripheral as below. You can place this just above the PWM configuration section from
the previous workshop.

```rust
let i2c_config = esp_hal::i2c::master::Config::default()
    .with_frequency(Rate::from_khz(50));
let i2c = I2c::new(peripherals.I2C0, i2c_config)
    .unwrap()
    .with_sda(peripherals.GPIO18)
    .with_scl(peripherals.GPIO19);
```
</details>

Next, we'll need to configure the touch driver reset GPIO, as it is required by the library.
You can find the pin number in the schematics.

<details>
<summary><strong>Solution</strong></summary>

```rust
let touch_driver_reset_pin = Output::new(
    peripherals.GPIO20,
    esp_hal::gpio::Level::Low,
    OutputConfig::default(),
);
```
</details>

And finally, create the instance of the touch driver and initialize it.

<details>
<summary><strong>Solution</strong></summary>

```rust
let mut touch_driver = axs5106l::Axs5106l::new(
    i2c,
    touch_driver_reset_pin,
    172,
    320,
    axs5106l::Rotation::Rotate0,
);

touch_driver.init(&mut delay).expect("failed to initialize the touch driver");
```
</details>

## Reading the data

The last part of the puzzle is to read out the data and print it out to the terminal.

<details>
<summary><strong>Solution</strong></summary>

The `get_touch_data()` function returns an Option wrapped in a Result. The Option indicates that there may not be
any data available (no touch event), while the Result is used to express any communication errors.

We can use the `match` statement to handle this in a simple way:

```rust
match touch_driver.get_touch_data() {
    Ok(Some(data)) => info!("{data:?}"),
    Ok(None) => (),
    Err(err) => error!("error reading touch data: {err:?}"),
}
```
</details>

## Exercise

For the final exercise, use the touch panel as a makeshift slider control to set the LED brightness.

<details>
<summary><strong>Solution</strong></summary>

```rust
const MAX_PWM_VALUE: u32 = 255;
const MAX_DISPLAY_Y: u32 = 320;

loop {
    match touch_driver.get_touch_data() {
        Ok(Some(data)) => {
            let brightness = (MAX_PWM_VALUE * data.points[0].y as u32 / MAX_DISPLAY_Y) as u16;
            info!("y={}, brightness={}", data.points[0].y, brightness);
            led.set_timestamp(brightness);
        }
        Ok(None) => (),
        Err(err) => error!("error reading touch data: {err:?}"),
    }

    delay.delay_millis(20);
}
```
</details>

[1]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0-rc.0/esp32c6/esp_hal/i2c/master/struct.I2c.html
[2]: setup/hardware.md
