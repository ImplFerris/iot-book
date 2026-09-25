{{#title Embedded Rust code for Building Burglar Alarm with ESP32-C5}}

# Code

Let's generate the project with `esp-generate` in headless mode.

```sh
esp-generate --headless -o esp32c5 -o defmt -o esp32c5-wroom-1-psram burglar-alarm
```

## Dependencies

We are going to change the color of the onboard LED to RED when motion is detected. To control the onboard LED, we need to add the following dependencies to `Cargo.toml`:

```toml
esp-hal-smartled = "0.18.0"
smart-leds = "0.4.0"
```

## Initializing the Onboard LED

We already saw this code snippet to initialize the onboard LED using the RMT peripheral in the previous section. We will use the same snippet here.

```rust
let freq = Rate::from_mhz(80);
let rmt = Rmt::new(peripherals.RMT, freq).unwrap();
let mut led =
    RmtSmartLeds::<{ buffer_size::<RGB8>(1) }, _, RGB8, color_order::Rgb>::new_with_memsize(
        esp_hal_smartled::WS2812_TIMING,
        rmt.channel0,
        peripherals.GPIO27,
        2,
        freq,
    )
    .unwrap();
```

## Initializing the PIR Sensor and Buzzer

Next, we will configure GPIO 10 to receive the signal from the PIR sensor. The PIR sensor sends its output signal through the `OUT` pin. When motion is detected, the sensor drives this pin HIGH. From the ESP32-C5's perspective, GPIO 10 is an input because it receives that signal. So, we will initialize this pin with the `Input` struct.

We will configure this pin with an internal pull-down resistor. This keeps the pin LOW when the PIR sensor is not driving it.

```rust
let sensor_pin = Input::new(
    peripherals.GPIO10,
    InputConfig::default().with_pull(Pull::Down),
);
```

Since the buzzer is controlled by the ESP32-C5, we will configure GPIO 24 as an output using the `Output` struct. We initialize it with `Level::Low` so that the buzzer remains off initially.

```rust
let mut buzzer_pin = Output::new(peripherals.GPIO24, Level::Low, OutputConfig::default());
```

## The Main Loop

The main loop is very simple. We will continuously check if we get a HIGH signal on the sensor pin. If so, we will turn on the buzzer and change the onboard LED to RED. If no motion is detected, we turn off both the buzzer and the LED.

```rust
loop {
    if sensor_pin.is_high() {
        info!("Motion detected");
        buzzer_pin.set_high();
        led.write([RGB8::new(255, 0, 0)]).unwrap();
    } else {
        buzzer_pin.set_low();
        led.write([RGB8::new(0, 0, 0)]).unwrap();
    }

    blocking_delay(Duration::from_millis(100));
}
```

The "Motion detected" message will be printed continuously while motion is detected. We could add a state flag to print the message only once until the motion stops, but I wanted to keep the code simple and easy to understand. You can extend this code to make it better.


## Clone the existing project

You can clone the project I created or refer to the existing project and navigate to the `burglar-alarm` folder.

```sh
git clone https://github.com/ImplFerris/esp32c5-projects
cd esp32c5-projects/burglar-alarm/
```

## Running the Project

Connect the ESP32-C5 to your computer and run the project:

```sh
cargo run --release
```

> [!NOTE]
> The PIR sensor needs some time to stabilize after power-up. Give it a short period before testing motion detection.

Once the project is running, move in front of the PIR sensor. When motion is detected, the buzzer will turn on and the onboard LED will change to red.

You might want to adjust the time delay and sensitivity according to your preference.
