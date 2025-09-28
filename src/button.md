# Exercise: working with inputs

We have worked through an example of driving a GPIO output.
Now it's time for you to try and extend the program to read the GPIO input state.

- Use the `esp_hal` crate documentation to create and initialize a GPIO [`Input`][1].
- The button is connected to `GPIO9`.
- Write a program that turns the LED on only while the button is pressed.

<details>
<summary><strong>Solution</strong></summary>

1. Add new imports:
   ```rust
   use esp_hal::gpio::{Input, InputConfig};
   ```
2. Declare the button:
   ```rust
   let button = Input::new(peripherals.GPIO9, InputConfig::default());
   ```
3. Modify the loop code:
   ```rust
    if button.is_high() {
        led.set_high();
    } else {
        led.set_low();
    }
   ```

### Explanation

You may be tempted to think that `high` means ON and `low` means off, however, the logic of both the button and the LED
is inverted.

Button has a pull-up resistor and connects the pin to ground when pressed.

The LED has its anode connected to the 3.3 V supply and the cathode is connected to the open-drain output. When the output is set to low, it is connected to ground, allowing the current to flow and thereby turning the LED on.

</details>

[1]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0-rc.0/esp32c6/esp_hal/gpio/struct.Input.html
