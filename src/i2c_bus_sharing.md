# I2C bus sharing

Before we dive into writing our IMU driver, we need to address some issues.

## Sharing the I2C bus

In the current setup, the touch panel driver is given an exclusive access to the I2C peripheral.
Our IMU is also on the I2C bus, so we have to share the peripheral between the two device drivers.

As explained in the [`embedded-hal`][1] crate documentation, we can use the [`embedded-hal-bus`][2] for this.
The crate provides several types of abstractions for bus sharing, bus since we're working in a single-threaded
application, a `RefCell` will be sufficient.

Let's start by importing the necessary types.

```rust
use core::cell::RefCell;
use embedded_hal_bus::i2c::RefCellDevice;
```

Then, right after `i2c` object instantiation, wrap the original object:

```rust
let i2c = RefCell::new(i2c);
```

Finally, in the instantiation of the touch driver, instead of passing `i2c` directly, create a new shared instance:

```rust
let mut touch_driver = axs5106l::Axs5106l::new(
    RefCellDevice::new(&i2c),
    ...
```

The code should compile and run just as before, give it a try.

[1]: https://docs.rs/embedded-hal/latest/embedded_hal/i2c/index.html#bus-sharing
[2]: https://docs.rs/embedded-hal-bus/latest/embedded_hal_bus/#i2c
