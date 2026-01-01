# Framebuffer

In the previous exercise, we saw that the display flickers during each redraw.
This is because we do perform two operations during each loop iteration:

1. we clear the screen
2. we send the new display state to the screen

Those familiar with graphics programming will already know the answer — we should use a [framebuffer][1].
A framebuffer is a block memory we use to store our display data.
We can clear and modify the buffer as many times as needed and only write it out to the display once the frame is complete.

As is usually the case in Rust, there's already a crate for that: [embedded-graphics-framebuf][2].
Let's add it to our project:

```sh
cargo add embedded-graphics-framebuf
```

Next, let's update our program to make use of it.

Import the necessary types. We will use `Rectangle` to draw the framebuffer on the screen.

```rust
use embedded_graphics_framebuf::FrameBuf;
use embedded_graphics::primitives::Rectangle;
```

Declare the memory array and create the framebuffer:

```rust
let mut data = [Rgb565::BLACK; DISPLAY_SIZE_W as usize * DISPLAY_SIZE_H as usize];
let mut frame_buffer =
    FrameBuf::new(&mut data, DISPLAY_SIZE_W as usize, DISPLAY_SIZE_H as usize);
```

Now, we can replace the `display` with `frame_buffer` in clear and draw operations.

The final step is to send the framebuffer contents to the display:

```rust
let area = Rectangle::new(Point::zero(), frame_buffer.size());
display.fill_contiguous(&area, frame_buffer.data.iter().copied()).ok();
```

[1]: https://en.wikipedia.org/wiki/Framebuffer
[2]: https://github.com/bernii/embedded-graphics-framebuf
