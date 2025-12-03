# Embedded graphics crate

We'll be starting our graphics journey using the [`embedded-graphics`][1] crate. This crate provides the functionality
for drawing basic shapes and text.

Let's start by putting a "Hellow, World!" text on the screen.

First, import the necessary types:

```rust
use embedded_graphics::{
    mono_font::{ascii::FONT_10X20, MonoTextStyle},
    text::Text,
};
```

Define the character style:

```rust
let character_style = MonoTextStyle::new(&FONT_10X20, Rgb565::BLACK);
```

And finally, draw the text:

```rust
Text::new(
    "Hello, World!",
    Point::new(90, DISPLAY_SIZE_H as i32 / 2),
    character_style,
)
.draw(&mut display)
.expect("could not draw text");
```

## Exercise

Make the text continuously scroll down, wrapping back to the top of the screen.

*Hint: check the `text.position` variable.*

<details>
<summary><strong>Solution</strong></summary>

```rust
let character_style = MonoTextStyle::new(&FONT_10X20, Rgb565::BLACK);

// 1. Make text a mutable variable, removing the call to draw().
let mut text = Text::new("Hello, World!", Point::new(90, 0), character_style);

// 2. Create a variable for storing the y position.
let mut y = 0;

loop {
    // 3. Clear the display every time, otherwise
    // the the text will just smudge on the screen.
    display.clear(Rgb565::WHITE).ok();

    // 4. Update text position and draw.
    text.position.y = y;
    text.draw(&mut display).ok();

    // 5. Handle y increment and wrapping.
    // y is at the baseline of the text, so add
    // the text height to make sure we scroll all the way.
    if y == DISPLAY_SIZE_H as i32 + 20 {
        y = 0;
    } else {
        y += 1;
    }

    delay.delay_millis(10);
}
```
> 💡 Note the use of `.ok()` in the example below.
>
> Rust compiler will warn us if we ignore the returned `Result` value,
> even though sometimes we really don't care if the operation failed.
>
> However, the compiler does not warn about unused `Option`, so we can use
> the `.ok()` method to convert the `Result` to `Option` and throw the value away.

</details>


[1]: https://github.com/embedded-graphics/embedded-graphics
