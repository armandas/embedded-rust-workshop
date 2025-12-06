# Testing IMU driver

Now that we got our basic IMU driver written, let's test it!

`cargo-generate` set's up our project as a library with a single binary application.
We therefore need to import our library, which has the same name as our project:

```rust
use hello_display::qmi8658a::Qmi8658a;
```

Next, we can create an instance of our IMU. Place this after the touch driver configuration:

```rust
// Configure IMU
let mut imu = Qmi8658a::new(RefCellDevice::new(&i2c), 0x6b);
```

Then, before the `loop {}`, read and print our the IMU ID:

```rust
match imu.read_chip_id() {
    Ok(id) => info!("IMU ID: {id:#04x}"),
    Err(err) => error!("failed to read IMU ID: {err}"),
}
```

If everything works as expected, you should see this output:

```
INFO - IMU ID: 0x05
```
