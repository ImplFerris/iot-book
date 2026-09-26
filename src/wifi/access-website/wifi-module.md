
# Wi-Fi Module

Let's start working with the Wi-Fi module. We need to provide the Wi-Fi credentials to connect to the network. Instead of hardcoding the credentials into the source code, we will load them from environment variables during compilation and store them in constants.

```rust
const SSID: &str = env!("SSID");
const PASSWORD: &str = env!("PASSWORD");
```

## Function to Initialize Wi-Fi

Next, we will create an `init_wifi` function. We will call this function from `main.rs` to initialize Wi-Fi. It takes the Embassy spawner and Wi-Fi peripheral as arguments, sets up everything needed for Wi-Fi, and returns the network stack handle.

```rust
pub async fn init_wifi(spawner: Spawner, wifi_peripheral: WIFI<'static>) -> Stack<'static> {
// ...
    stack
}
```

Let's begin by creating the Wi-Fi station configuration. We will use WPA2 (Wi-Fi Protected Access 2) for authentication, which is commonly used for password-protected Wi-Fi networks.

```rust
let station_config = wifi::Config::Station(
    StationConfig::default()
        .with_ssid(SSID.try_into().unwrap())
        .with_authentication(AuthenticationMethodConfig::Wpa2Personal(
            PASSWORD.try_into().unwrap(),
        )),
);
```

Next, we create the Wi-Fi controller using the station configuration.

```rust
let mut controller = esp_radio::wifi::WifiController::new(
    wifi_peripheral,
    wifi::ControllerConfig::default().with_initial_config(station_config),
)
.expect("Failed to initialize Wi-Fi controller");
```

Next, we create the network stack by calling `embassy_net::new`, which returns the network stack handle along with the network stack runner.

```rust
let rng = Rng::new();
let seed = (rng.random() as u64) << 32 | rng.random() as u64;

let config = embassy_net::Config::dhcpv4(Default::default());
let wifi_interface = esp_radio::wifi::Interface::station();

// Init network stack
let (stack, runner) = embassy_net::new(
    wifi_interface,
    config,
    mk_static!(StackResources<3>, StackResources::<3>::new()),
    seed,
);
```

Next we spawn two Embassy tasks. The `connection` task handles the Wi-Fi connection, while the `net_task` processes network events. We will define these two functions shortly.

```rust
spawner.spawn(connection(controller).unwrap());
spawner.spawn(net_task(runner).unwrap());
```

Now we wait for the network configuration to be ready. Once we get an IPv4 address, we print it.

```rust
stack.wait_config_up().await;

if let Some(config) = stack.config_v4() {
    info!("Got IP: {}", config.address);
}
```

This may not be needed for you. The `esp-hal` example does not set this, but in my case, I got a DNS error, so I set the DNS entries manually. We will also define this function shortly.

```rust
set_dns_servers(stack);
```

With this setup, we have reached the end of the `init_wifi` function. At the end of the function, we return the `stack` variable.

## Set DNS Servers

I will be using Google and Cloudflare DNS servers.

```rust
fn set_dns_servers(stack: Stack<'static>) {
    // Custom DNS Servers
    if let Some(mut cfg) = stack.config_v4() {
        cfg.dns_servers.copy_from_slice(&[
            embassy_net::Ipv4Address::new(1, 1, 1, 1),
            embassy_net::Ipv4Address::new(8, 8, 8, 8),
        ]);

        stack.set_config_v4(embassy_net::ConfigV4::Static(cfg));
        info!("Overrode DNS servers, keeping DHCP address/gateway");
    }
}
```

## Network Task

This task runs the network stack in the background and processes network events.

```rust
#[embassy_executor::task]
async fn net_task(mut runner: Runner<'static, Interface>) {
    runner.run().await
}
```

## Wi-Fi Connection Task

This task handles connecting to the Wi-Fi network and reconnecting when the connection is lost.

```rust
#[embassy_executor::task]
async fn connection(mut controller: WifiController<'static>) {
    info!("start connection task");

    loop {
        info!("About to connect...");

        match controller.connect_async().await {
            Ok(info) => {
                info!("Wifi connected to {:?}", info);

                // wait until we're no longer connected
                let info = controller.wait_for_disconnect_async().await.ok();
                info!("Disconnected: {:?}", info);
            }
            Err(e) => {
                info!("Failed to connect to wifi: {:?}", e);
            }
        }

        Timer::after(Duration::from_millis(5000)).await
    }
}
```
