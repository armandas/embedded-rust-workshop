# Displaying formatted text

So far, we've been happily printing formatted text to the console, but how do we do that on the LCD screen?

## `format!`

The Rust standard library's [`alloc` crate][1] provides the [`format!` macro][2], which gives us a `String` output.
Let's try it quickly.

Add the `alloc` crate and import the `format` macro.

```rust
extern crate alloc;
use alloc::format;
```

Then add this code to the main loop, after `frame_buffer.clear()`:

```rust
let message = format!(
    "Current timestamp: {} ms",
    Instant::now().duration_since_epoch().as_millis()
);
let text2 = Text::new(&message, Point::new(30, y + 20), character_style);
text2.draw(&mut frame_buffer).ok();
```

If we try to compile our program, we'll get an error:

```
error: no global memory allocator found but one is required; link to std or add `#[global_allocator]` to a static item that implements the GlobalAlloc trait
```

In order to create a `String`, we need dynamic memory allocation (hence the `alloc` crate).
However, on embedded systems, dynamic memory allocation is often not desirable and is not enabled by default.

For ESP32 projects, we can enable it at project creation time using `esp-generate`,
but we can also set it up after the fact by adding the [`esp-alloc` crate][3]:

```sh
cargo add esp-alloc --features esp32c6
```

Lastly, declare the allocator by adding this line near the beginning of the `main` function.

```rust
esp_alloc::heap_allocator!(size: 1024);
```

The size can be adjusted based on project needs, but we just want to format a short string, so 1 kB should be plenty!

If we run our program now, the message should successfully show up on the screen.


[1]: https://doc.rust-lang.org/nightly/alloc/index.html
[2]: https://doc.rust-lang.org/nightly/alloc/macro.format.html
[3]: https://docs.espressif.com/projects/rust/esp-alloc/0.9.0/esp_alloc/index.html
