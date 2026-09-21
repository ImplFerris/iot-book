# Understanding the Project

We will quickly go through the Quick Start project and look at some of the basics behind the code and tools we are using.

## Dependencies

When you generate the project with the `esp-generate` tool, the following dependencies gets added.

The `esp-hal` supports many different ESP32 variants. The code for each variant is separated using features. For our ESP32-C5, we need to enable the "esp32c5" feature.

The `esp-bootloader-esp-idf` crate contains functionality related to the ESP-IDF second-stage bootloader. Once again, we need to enable the "esp32c5" feature for this also.

The `critical-section` crate provides a common way for embedded libraries to create critical sections. A critical section is used to protect code or data from being accessed concurrently.

The `defmt` crate provides a logging framework designed for resource-constrained devices such as microcontrollers. We used it in the Quick Start program to print the `Hello world!` message.

## `no_std` and `no_main`

We use the `#![no_std]` attribute to indicate that we won't be using Rust's standard library. The `#![no_main]` attribute tells Rust that we are not using the standard `main` entry point provided by the standard Rust environment.

```rust
#![no_std]
#![no_main]
```

## Panic Handler

In a typical Rust application, the standard library provides a default panic handler. Since we are not using the standard library, we need to define our own panic handler.

```rust
#[panic_handler]
fn panic(panic_info: &core::panic::PanicInfo) -> ! {
    error!("{}", panic_info);
    loop {}
}
```

## Main entrypoint

The `#[main]` attribute comes from esp-hal and marks our entry point:

```rust
#[main]
fn main() -> ! {
    //...
}
```

The `!` return type means the function never returns, which is normal for embedded firmware that keeps running.

## Peripherals

The `peripherals` variable gives us access to the ESP32-C5's peripherals, such as GPIO, ADC, I2C, and SPI.

```rust
let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
let peripherals = esp_hal::init(config);
```

## Flashing

Normally, when you run `cargo run`, Cargo builds and runs the program on your system. In our case, `cargo run` builds the program for the ESP32-C5 and flashes the firmware to the microcontroller. This was achieved with the help of the `.cargo/config.toml` file.

The actual tool used to flash the firmware is `espflash`. The program gets compiled for the `riscv32imac-unknown-none-elf` target, and the runner is configured to use `espflash`.

This makes the flashing process much easier. Instead of running the full command with all the required options, we can simply run `cargo run`.
