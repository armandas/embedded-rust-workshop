# Basic IMU driver

## Preparation

We'll start by creating a file in our `src/` directory named `qmi8658a.rs`.

Then, export it in the `lib.rs`:

```rust
pub mod qmi8658a;
```

The next thing we'll need for our library is the `embedded-hal` traits, so let's add them now:

```sh
cargo add embedded-hal
```

## Defining the driver struct

Back in the `qmi8658a.rs`, we already know we'll need the `I2c` trait, so let's import it first:

```rust
use embedded_hal::i2c::I2c;
```

We can now start writing our driver by declaring the struct. To start with, we know we'll need to keep around two things:
I2C instance and the I2C address of our device.

Some I2C devices have a fixed address, however, many allow the address to be configured by pulling a GPIO high or low.
This allows multiple devices to be connected on the same bus. In case of our IMU, the address can be either `0x6A` or
`0x6B` depending if IMU pin 1 is pulled high or low.

Our board pulls the GPIO low, so in effect the address is fixed, we'll make the driver reusable across different
designs by taking the address as a parameter.

```rust
pub struct Qmi8658a<I2C: I2c> {
    i2c: I2C,
    address: u8,
}
```

You'll notice that our struct has a generic parameter `I2C`. Similar to templates in C++, it allows us to take
different concrete types without knowing about them in advance.

The `: I2c` part is a [trait bound][1]. It restricts the types of `I2C` to only those concrete types that implement the
`embedded_hal::i2c::I2c` trait.


## Constructor

Next, we'll start adding methods for our driver. The first one is the `new()` method:

```rust
impl<I2C: I2c> Qmi8658a<I2C> {
    pub fn new(i2c: I2C, address: u8) -> Self {
        Self { i2c, address }
    }
}
```

## Reading the chip ID

Now we're getting to the exciting part: reading data from the IMU via the I2C bus.
We'll start easy, by reading the fixed chip ID. The fact that the ID is fixed will allow us to say with
confidence if our code is working or not.

We don't like magic numbers, so let's define our register addresses.

```rust
mod registers {
    pub const WHO_AM_I: u8 = 0x00;
}
```

Now we can add a new method to our `impl` block:

```rust
pub fn read_chip_id(&mut self) -> Result<u8, I2C::Error> {
    let mut id = [0];
    self.i2c.write_read(self.address, &[registers::WHO_AM_I], &mut id)?;
    Ok(id[0])
}
```



[1]: https://doc.rust-lang.org/rust-by-example/generics/bounds.html
