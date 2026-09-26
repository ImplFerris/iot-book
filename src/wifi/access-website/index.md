{{#title Connect ESP32-C5 to a Wi-Fi Network Using Rust}}

# Connect ESP32-C5 to a Wi-Fi Network Using Rust

In this exercise, we will connect the ESP32-C5 to an existing Wi-Fi network in Station mode using Rust. Once connected, we will query a website, fetch some data, and print it to the system console.

## Prerequisites

For this exercise, you will need a Wi-Fi network. You can use either your home Wi-Fi network or a mobile hotspot. Note down the SSID (Wi-Fi network name) and its password.

## Project Setup

With `esp-generate`, enable "Unstable HAL" because the Wi-Fi APIs are currently unstable. Also enable "alloc", which is required by the Wi-Fi implementation.  The Wi-Fi support also requires the Embassy, so we need to enable the Embassy option as well.

Run the following command to create the `wifi-client` project:

```sh
esp-generate --headless \
  -o esp32c5 \
  -o esp32c5-wroom-1-psram \
  -o unstable-hal \
  -o alloc \
  -o embassy \
  -o wifi \
  -o defmt \
  access-website
```

## Wi-Fi Boilerplate Code

The `esp-generate` tool generates the following boilerplate code when the Wi-Fi option is enabled. This code provides the basic Wi-Fi setup, but we need to modify it to initialize Wi-Fi in station mode and provide the Wi-Fi credentials.

```rust
esp_alloc::heap_allocator!(#[esp_hal::ram(reclaimed)] size: 65536);

let timg0 = TimerGroup::new(peripherals.TIMG0);
esp_rtos::start(timg0.timer0, peripherals.FROM_CPU_INTR0);

info!("Embassy initialized!");

let _wifi_controller =
    esp_radio::wifi::WifiController::new(peripherals.WIFI, Default::default())
        .expect("Failed to initialize Wi-Fi controller");
let _wifi_interface = esp_radio::wifi::Interface::station();

// TODO: Spawn some tasks
let _ = spawner;

loop {
    info!("Hello world!");
    Timer::after(Duration::from_secs(1)).await;
}
```

## Project structure

The Wi-Fi initialization setup will be too big and I don't like it to live in the `main.rs` file. So instead we will create a separate `wifi.rs` module where we will initialize the Wi-Fi and do the necessary boilerplate setup, then return the Wi-Fi stack. In this way, we can easily copy the `wifi.rs` module to other projects and use it.

The project structure will look like this:

```sh
├── src
│   ├── bin
│   │   └── main.rs
│   ├── lib.rs
│   └── wifi.rs
```

## Additional Crate

Add the following dependencies to your `Cargo.toml`:

```toml
reqwless = { version = "0.14.0", default-features = false, features = [
    "defmt",
    ] }
```

We will use the `reqwless` crate to send HTTP requests. It provides an HTTP client that can be used in a `no_std` environment with any transport that implements the traits from the `embedded-io` crate. It does not require `alloc`.

## Using StaticCell

Some values in our program are created at runtime but need to live for the entire lifetime of the program. For this, we will use `StaticCell`, which allows us to initialize these values at runtime and get a `&'static mut` reference to them. We use the `mk_static!` macro provided in the `esp-hal` examples to make this easier to reuse.

Filename: src/lib.rs

```rust
#![no_std]

pub mod wifi;

#[macro_export]
macro_rules! mk_static {
    ($t:ty,$val:expr) => {{
        static STATIC_CELL: static_cell::StaticCell<$t> = static_cell::StaticCell::new();
        #[deny(unused_attributes)]
        let x = STATIC_CELL.uninit().write($val);
        x
    }};
}
```

In addition, you create the `wifi.rs` module and add it to `lib.rs`, which we will work on in the next section.
