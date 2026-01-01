# GPIO Interrupt

Let's see how we can use interrupts on ESP32.

## Declaring global mutable objects

At least in case of `esp_hal`, interrupt status is not cleared automatically, so we'll need to do that ourselves.
For that, we will need to access the button object inside the interrupt handler, so we have to make it global.

Additionally, since we will be modifying this global object, we have to ensure it's done in a *safe* way, without
causing race conditions.

We'll need to wrap our `Input` with three different types, so this will be very confusing at first. The purpose of each
type will be explained at the end of this section.

Start by importing the necessary types:

```rust
use core::cell::RefCell;
use critical_section::Mutex;
```

Now, we can declare our global button:

```rust
static BUTTON: Mutex<RefCell<Option<Input>>> = Mutex::new(RefCell::new(None));
```

As you can see, our `Input` is wrapped in an `Option`, which is wrapped in a `RefCell` which in turn is wrapped
in `Mutex`.

Let's look into each one starting with the outermost.

### Mutex

[`Mutex`][3], when combined with the critical section, allows us to get an exclusive access to the stored type.
Unlike the `Mutex` in Rust's `std` library, `critical_section::Mutex` does not provide ability to mutate the
contained value. For this, we have to rely on the next type.

### RefCell

In short, [`RefCell`][4] is a run-time borrow checker. When we call `borrow` or `borrow_mut` methods, the type will check
that the value is not already borrowed. If it is, we will get a panic. If we successfully get the mutable reference,
we can safely mutate the stored value, because we know that no-one else has any reference to our object.

### Option

[`Option`][5] is required because we cannot create `Input` at the initialization time. We need to initialize the chip,
configure peripherals etc. `Option` allows us to initialize an "empty" object first and fill it later when we're ready.

## Handler function

Next, let's look how we can define an interrupt handler function.

Import the `handler` macro:

```rust
use esp_hal::handler;
```

Then define the handler function at the bottom of the file, like this:

```rust
#[handler]
fn button_handler() {
    info!("GPIO interrupt!");
    critical_section::with(|cs| {
        BUTTON
            .borrow(cs)         // Borrow the value in Mutex
            .borrow_mut()       // Mutably borrow value in RefCell
            .as_mut()           // Get mutable reference to the value in Option
            .unwrap()           // Unwrap the Option<&mut T>
            .clear_interrupt(); // Clear the interrupt flag
    });
}
```

> 💡 To make things a bit more concise, `Mutex` provides a `borrow_ref_mut` method that combines the `Mutex` borrow
> and `RefCell` borrow into one function. We will use that function from here on.

> 💡 Note how our handler function is marked with the [`#[handler]`][1] attribute macro.
> We can use [`cargo-expand`][2] tool to expand the macros and see what code is being produced.
> Our handler function gets expanded to the code below:
> ```rust
> extern "C" fn __esp_hal_internal_button_handler() {
>     // Function body goes here
> }
>
> #[allow(non_upper_case_globals)]
> const button_handler: esp_hal::interrupt::InterruptHandler =
>     esp_hal::interrupt::InterruptHandler::new(
>         __esp_hal_internal_button_handler,
>         esp_hal::interrupt::Priority::min(),
>     );
> ```

## Setting up interrupts

Next, we can register our handler to handle the GPIO interrupts. Add this to the setup part of your `main` function:

```rust
let mut io = esp_hal::gpio::Io::new(peripherals.IO_MUX);
io.set_interrupt_handler(button_handler);
```

Finally, we start to listen to the button events and move the button object into our global state.
We will use a "rising edge" event, which means that the interrupt will be triggered on button release.

Import `Event`:

```rust
use esp_hal::gpio::Event;
```

Add this code right after the existing declaration of `button`.

```rust
critical_section::with(|cs| {
    button.listen(Event::RisingEdge);
    BUTTON.borrow_ref_mut(cs).replace(button);
});
```

## Experiment

Try changing the event type to falling edge. Do you observe any difference?


[1]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0-rc.0/esp32c6/esp_hal/attr.handler.html
[2]: https://github.com/dtolnay/cargo-expand
[3]: https://docs.rs/critical-section/latest/critical_section/struct.Mutex.html
[4]: https://doc.rust-lang.org/nightly/std/cell/struct.RefCell.html
[5]: https://doc.rust-lang.org/nightly/std/option/index.html
