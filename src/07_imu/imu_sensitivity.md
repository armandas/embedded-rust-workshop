# Exercise: configure IMU sensitivity

By default, the full scale of the accelerometer is set to ±2 g and the full scale of the gyro is only ±16 dps
(degrees per second).

If the accelerometer moves faster than those limits, the data will be clamped, resulting in an incorrect measurement.

Look at the datasheet to find the correct registers to modify and implement the necessary functions for the `Config`
struct.

<details>
<summary><strong>Solution</strong></summary>

### Step 1: update the boilerplate

Registers `CTRL2` and `CTRL3` contain settings for accelerometer and gyro respectively.
Let's add those to our `Config` struct.

```rust
#[derive(Debug)]
pub struct Config {
    ctrl1: u8,
    ctrl2: u8,
    ctrl3: u8,
    ctrl7: u8,
}
```

Define the addresses:

```rust
mod registers {
    // ...
    pub const CTRL2: u8 = 0x03;
    pub const CTRL3: u8 = 0x04;
    // ...
}
```

Update the `initialize()` function to write the new config registers:

```rust
self.i2c.write(self.address, &[registers::CTRL2, config.ctrl2])?;
self.i2c.write(self.address, &[registers::CTRL3, config.ctrl3])?;
```

### Step 2: define types for `Config`

As the range can only be one of several discrete settings, we'll use enums to represent the values.

Accelerometer range can be configured from 2 g to 16 g.

```rust
pub enum AccelRange {
    PlusMinus2g,
    PlusMinus4g,
    PlusMinus8g,
    PlusMinus16g,
}
```

The gyro range can be configured from 16 dps to 2048 dps.

```rust
pub enum GyroRange {
    PlusMinus16Dps,
    PlusMinus32Dps,
    PlusMinus64Dps,
    PlusMinus128Dps,
    PlusMinus256Dps,
    PlusMinus512Dps,
    PlusMinus1024Dps,
    PlusMinus2048Dps,
}
```

### Step 3: implement the config functions

Here's how you can implement the config function.
We use `match` to convert the enum values to the corresponding binary value obtained from the datasheet.

```rust
pub fn with_accel_full_scale_range(mut self, range: AccelRange) -> Config {
    let bits = match range {
        AccelRange::PlusMinus2g => 0b000,
        AccelRange::PlusMinus4g => 0b001,
        AccelRange::PlusMinus8g => 0b010,
        AccelRange::PlusMinus16g => 0b011,
    };

    self.ctrl2 |= bits << 4;
    self
}

pub fn with_gyro_full_scale_range(mut self, range: GyroRange) -> Config {
    let bits = match range {
        GyroRange::PlusMinus16Dps => 0b000,
        GyroRange::PlusMinus32Dps => 0b001,
        GyroRange::PlusMinus64Dps => 0b010,
        GyroRange::PlusMinus128Dps => 0b011,
        GyroRange::PlusMinus256Dps => 0b100,
        GyroRange::PlusMinus512Dps => 0b101,
        GyroRange::PlusMinus1024Dps => 0b110,
        GyroRange::PlusMinus2048Dps => 0b111,
    };

    self.ctrl3 |= bits << 4;
    self
}
```

### Step 4: update the `main.rs`

Use the new configuration functions:

```rust
let imu_config = qmi8658a::Config::default()
    .with_address_auto_increment()
    .with_accel_full_scale_range(qmi8658a::AccelRange::PlusMinus16g)
    .with_gyro_full_scale_range(qmi8658a::GyroRange::PlusMinus2048Dps)
    .with_gyroscope_enabled()
    .with_accelerometer_enabled()
    .with_sync_sample_enabled();
```

</details>

