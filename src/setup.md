{{#title ESP32-C5 Development Setup for Embedded Rust}}

# Development Environment

The [official docs](https://docs.espressif.com/projects/rust/book/getting-started/index.html) provides more comprehensive setup instructions. However, I will quickly cover the essential tools and setup needed for our exercises. If you encounter any issues, refer to the official documentation for troubleshooting.

## Rust toolchain

The ESP32-C5 uses a 32-bit RISC-V processor, so we can use the standard Rust toolchain. We do not need `espup`, which is used for ESP chips that require the Xtensa Rust toolchain.

Install the required Rust components:
```sh
rustup component add rust-src
rustup target add riscv32imac-unknown-none-elf
```
 
## cargo-binstall

This is to install Rust binaries without building from source using cargo install or manually downloading packages, you can use cargo-binstall. We'll use this tool to install the espflash tool next.

```sh
cargo install cargo-binstall
```

## espflash

"espflash is a serial flasher utility, based on esptool.py, for Espressif SoCs and modules."  This will be the tool used (when we are not using probe-rs) to put our code into the device and run it. 

```sh
cargo binstall espflash
```

If you encounter any problems, you can try installing the exact version used when this book was written.

```sh
cargo binstall espflash@4.6.0
```

After installation, type the espflash command to verify that it works.

```sh
espflash --version
```

## Template by ESP-RS

We will be using the templates provided by [ESP-RS](https://docs.espressif.com/projects/rust/book/getting-started/tooling/esp-generate.html), which offer two sets:  

- **esp-generate**: A `no_std` template. This is the one we will focus on most of the time.  
- **esp-idf-template**: A `std` template.

## esp-generate
The **esp-generate** tool is used for creating `no_std` applications. Currently, it supports the ESP32, ESP32-C2/C3/C6, ESP32-H2, and ESP32-S2/S3. 

```sh
cargo install esp-generate --locked
```

If you want to follow the code exactly as it is in this project, install this esp-generate version used for generating the examples:
```sh
cargo install esp-generate@1.4.0 --locked
```


### Creating project with `esp-generate`

For this book, we will be using the ESP32-C5. I highly recommend using the same hardware to make it easier to follow along.

```sh
esp-generate PROJECT_NAME
```

## USB Access

On Linux, your user needs permission to access the USB serial port used by the ESP32-C5. Add your user to the `dialout` group:

```sh
sudo usermod -a -G dialout $USER
```

Log out and back in for the changes to take effect.
