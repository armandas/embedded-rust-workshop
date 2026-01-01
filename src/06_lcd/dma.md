# DMA

One more trick to gain a little bit of performance is to use [Direct Memory Access][1] to write data out to the LCD.
We'll make use of the [SPI DMA][2] support in the `esp-hal` crate.

As usual, let's start by importing the necessary types.

```Rust
use esp_hal::{dma::{DmaRxBuf, DmaTxBuf}, dma_buffers};
```

Next, we need to declare the DMA buffers:

```Rust
let (rx_buffer, rx_descriptors, tx_buffer, tx_descriptors) = dma_buffers!(10 * 1024);
let dma_rx_buf = DmaRxBuf::new(rx_descriptors, rx_buffer).expect("failed to create DMA RX buffer");
let dma_tx_buf = DmaTxBuf::new(tx_descriptors, tx_buffer).expect("failed to create DMA TX buffer");
```

Finally, update the SPI builder with calls to `with_dma()` and `with_buffers()` and you're done!

```Rust
let spi = Spi::new(
    peripherals.SPI2,
    spi::master::Config::default()
        .with_frequency(Rate::from_mhz(80))
        .with_mode(spi::Mode::_0),
)
.expect("could not create SPI instance")
.with_sck(peripherals.GPIO1)
.with_mosi(peripherals.GPIO2)
.with_dma(peripherals.DMA_CH0)         // New!
.with_buffers(dma_rx_buf, dma_tx_buf); // New!
```

## Exercise

Can you measure the time difference between regular and DMA-enabled implementations?

[1]: https://en.wikipedia.org/wiki/Direct_memory_access
[2]: https://docs.espressif.com/projects/rust/esp-hal/1.0.0/esp32c6/esp_hal/spi/master/struct.SpiDma.html
