{{#title ESP32-C5 Rust Quick Start - Getting Started Program}}

# Quick Start - Hello Embedded!

Before getting into other concepts, let's quickly check that our development setup is working and run our first Rust program on the ESP32-C5.

### Setup project

To start the project, use the `esp-generate` command. Run the following:

```sh
esp-generate esp32c5-quick
```

This will open the configuration menu.

First, select `ESP32-C5` as the target chip. Next, select `ESP32-C5-WROOM-1/1U (8MB PSRAM)` as the chip variant. In the "Flashing, logging and debugging (espflash)" section, enable defmt messaging.

For the remaining options, we will use the default settings. Save the configuration to generate the project. Just press s on the keyboard.

Alternatively, you can use the following command without using the TUI. This comes in handy when you already know all the options you want to use.

```sh
esp-generate --headless -o esp32c5 -o defmt -o esp32c5-wroom-1-psram esp32c5-quick
```

## Program

I usually introduce a simple blinky program in the Quick Start section of my other books. However, in the case of the ESP32-C5 board, we can't do that because the board has an addressable RGB LED instead of a standard LED. It cannot be controlled by simply toggling a GPIO pin between High and Low.

So, for this Quick Start, we will simply print a message using defmt.

We don't need to make any changes to the generated program. We will only increase the delay between messages from 500 milliseconds to 5 seconds.

```rust
#[allow(
    clippy::large_stack_frames,
    reason = "it's not unusual to allocate larger buffers etc. in main"
)]
#[main]
fn main() -> ! {
    // generator version: 1.4.0
    // generator parameters: -o esp32c5 -o esp32c5-wroom-1-psram -o defmt

    let config = esp_hal::Config::default().with_cpu_clock(CpuClock::max());
    let _peripherals = esp_hal::init(config);

    loop {
        info!("Hello world!");
        let delay_start = Instant::now();
        while delay_start.elapsed() < Duration::from_millis(5000) {}
    }
}
```

## Flash - `Run Rust Run`

All that's left is to flash the code onto the ESP32-C5 and see the output.

Connect the ESP32-C5 to your computer using a USB-C cable.

Then, run the following command from your project folder:

```rust
cargo run
```

To run in release mode:

```rust
cargo run --release
```

Once the program starts, you should see the `Hello world!` message printed every five seconds.
