# IMU configuration

Now that we've confirmed communication with the IMU, we can proceed to the next step.
The IMU starts with all sensors disabled, so in order to take any measurements, we need to configure the chip.

There are other useful settings that we'll want to change, but for now, we'll focus on the most critical ones
in registers `CTRL1` and `CTRL7`.

We will use a configuration builder similar to those provided by `esp-hal` peripherals.
Let's start by defining our struct:

```rust
#[derive(Debug)]
pub struct Config {
    ctrl1: u8,
    ctrl7: u8,
}
```

We'll implement the `Default` trait, so that we can create an instance of our `Config`.
We're using the default values from the datasheet.

```rust
impl Default for Config {
    fn default() -> Self {
        Self {
            ctrl1: 0b0010_0000,
            ctrl7: 0b0000_0000,
        }
    }
}
```

Now that we've got the default configuration, we need to provide a way to change the settings.
Let's look at a few functions as an example:

```rust
impl Config {
    // CTRL1
    pub fn with_3_wire_spi(mut self) -> Self {
        self.ctrl1 |= 1 << 7;
        self
    }

    pub fn with_address_auto_increment(mut self) -> Self {
        self.ctrl1 |= 1 << 6;
        self
    }

    pub fn with_little_endian_data(mut self) -> Self {
        // Note: the bit is set by default, so we provide an API to clear it.
        self.ctrl1 &= !(1 << 5);
        self
    }

    // ...
}
```

Implement the remaining methods for `CTRL1` and `CTRL7` registers.

## IMU initialization

In order to make use of the configuration, we need to be able to write the data to the chip.
Let's add an `initialize()` method to our IMU instance to do just that:

```rust
pub fn initialize(&mut self, config: Config) -> Result<(), I2C::Error> {
    self.i2c.write(self.address, &[registers::CTRL1, config.ctrl1])?;
    self.i2c.write(self.address, &[registers::CTRL7, config.ctrl7])?;
    Ok(())
}
```

## Bringing it all together

Finally, in the `main.rs`, we can create our configuration and initialize the IMU:

```rust
let imu_config = qmi8658a::Config::default()
    .with_address_auto_increment()
    .with_gyro_enabled()
    .with_accelerometer_enabled();
```

And then pass the `imu_config` to the `initialize()` function:

```rust
imu.initialize(imu_config).expect("failed to initialize IMU");
```
