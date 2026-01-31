# Reading IMU data

The last missing piece now is to read the actual IMU data. Because the IMU data is made up of several registers
and they are constantly being updated by the hardware, we need to take some steps to avoid data corruption.

The steps are outlined in the page 70 of the datasheet:

1. Configure the IMU with SyncSample setting on.
2. To begin reading out the data, first read the `STATUSINT` register.
3. If the `Avail` bit is 0, go back to step 2.
4. Check the `Locked` bit. If it is 0, we need to wait up to 270 us before proceeding.
5. Read out the IMU registers.

## Passing in the `delay`

Since we need a delay functionality, let's first update our IMU struct to take a delay instance as a construction parameter.
Similarly to the I2C, we will use a trait from the `embedded_hal` crate:

```rust
use embedded_hal::delay::DelayNs;
```

Modify out IMU struct:

```rust
#[derive(Debug)]
pub struct Qmi8658a<I2C: I2c, Delay: DelayNs> {
    i2c: I2C,
    address: u8,
    delay: Delay,
}
```

And finally, the `impl` block and the `new` function:

```rust
impl<I2C: I2c, Delay: DelayNs> Qmi8658a<I2C, Delay> {
    pub fn new(i2c: I2C, address: u8, delay: Delay) -> Self {
        Self { i2c, address, delay }
    }

    // ...
```

## Updating registers

We also need to update out register list to include the `STATUSINT` and the first register of the IMU data, `AX_L`:

```rust
mod registers {
    pub const WHO_AM_I: u8 = 0x00;
    pub const CTRL1: u8 = 0x02;
    pub const CTRL7: u8 = 0x08;
    pub const STATUSINT: u8 = 0x2d;
    pub const TEMP_L: u8 = 0x33;
    pub const AX_L: u8 = 0x35;
}
```

## Reading the status register

To make things easier later on, let's write a function that reads and returns the value of the `STATUSINT` register.

```rust
pub fn read_status_int(&mut self) -> Result<u8, I2C::Error> {
    let mut status_int = [0; 1];
    self.i2c
        .write_read(self.address, &[registers::STATUSINT], &mut status_int)?;
    Ok(status_int[0])
}
```

## Reading out the IMU data

To start with, we need a way to return the data. Let's create a struct for that:

```rust
#[derive(Debug)]
pub struct ImuData {
    pub accel_x: i16,
    pub accel_y: i16,
    pub accel_z: i16,
    pub gyro_x: i16,
    pub gyro_y: i16,
    pub gyro_z: i16,
}
```

Next, let's define bit positions of the `STATUSINT` register:

```rust
const STATUS_INT_AVAIL: u8 = 1 << 0;
const STATUS_INT_LOCKED: u8 = 1 << 1;
```

Finally, let's create the data readout function. We'll start with the signature and fill in the body step-by-step.

```rust
pub fn read_imu_data(&mut self) -> Result<Option<ImuData>, I2C::Error> {
    todo!()
}
```

A quick note on the return type. We're returning an `Option` wrapped in a `Result`.
The result indicated the success of the I2C read operations, while the `Option` tells us if the data was available or not.

Now, let's implement the steps discussed at the beginning.
Steps 2 and 3: read the `STATUSINT` register and check if the data is available:

```rust
let status_int = self.read_status_int()?;
if (status_int & STATUS_INT_AVAIL) != STATUS_INT_AVAIL {
    // Data not available yet
    return Ok(None);
}
```

Step 4: check the locked bit and wait if necessary:

```rust
if (status_int & STATUS_INT_LOCKED) != STATUS_INT_LOCKED {
    // Wait until data is locked
    self.delay.delay_us(270);
}
```

Step 5: read the data and construct the return value:

```rust
let mut imu = [0; 12];
self.i2c
    .write_read(self.address, &[registers::AX_L], &mut imu)?;

Ok(Some(ImuData {
    accel_x: i16::from_le_bytes(imu[0..2].try_into().unwrap()),
    accel_y: i16::from_le_bytes(imu[2..4].try_into().unwrap()),
    accel_z: i16::from_le_bytes(imu[4..6].try_into().unwrap()),
    gyro_x: i16::from_le_bytes(imu[6..8].try_into().unwrap()),
    gyro_y: i16::from_le_bytes(imu[8..10].try_into().unwrap()),
    gyro_z: i16::from_le_bytes(imu[10..12].try_into().unwrap()),
}))
```

## Testing

To test, let's update the `imu` construction and enable `SyncSample` mode in the `main.rs`:

```rust
let mut imu = Qmi8658a::new(RefCellDevice::new(&i2c), 0x6b, delay.clone());

let imu_config = qmi8658a::Config::default()
    .with_address_auto_increment()
    .with_gyroscope_enabled()
    .with_accelerometer_enabled()
    .with_sync_sample_enabled();
```

Then in the main loop, we can read and print our data:

```rust
if let Ok(Some(data)) = imu.read_imu_data() {
    info!("{data:?}");
}
```
