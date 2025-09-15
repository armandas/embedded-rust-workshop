# Setup

## 1. Rust

Go to [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)

> 💡 Windows users: prefer to install Rust natively, rather than using WSL.

## 2. Toolchain

To compile Rust code for out ESP32-C6 microcontroller, we need to install the appropriate toolchain.

```text
rustup toolchain install stable --component rust-src --target riscv32imac-unknown-none-elf
```

## 3. Tools

Additionally, we will want tools to help us generate projects and program the firmware.

```text
cargo install cargo-espflash espflash esp-generate
```
