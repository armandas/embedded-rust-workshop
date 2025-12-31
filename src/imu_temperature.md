# Reading IMU temperature

With the chip configured, we can take the next step and read out the IMU temperature.

The temperature data is contained in two registers, the high byte holding the degree part and
the low byte holding the fractional part. We'll combine the two parts into a single `i16` value, which
the user can then divide by 256 to get the temperature in a floating-point format.

```rust
pub fn read_temperature(&mut self) -> Result<i16, I2C::Error> {
    let mut temperature = [0; 2];
    self.i2c.write_read(self.address, &[registers::TEMP_L], &mut temperature)?;
    Ok(i16::from_le_bytes(temperature))
}
```

> 💡 Note that we're making use of the address auto-increment feature. We  send only `TEMP_L` address,
> but we can read out both `TEMP_L` and `TEMP_H` by simply reading two bytes.

Now, we can add temperature print-out to our main loop:

```rust
if let Ok(temperature) = imu.read_temperature() {
    info!("Temperature: {:#06X} = {}", temperature, temperature as f32 / 256f32);
}
```
