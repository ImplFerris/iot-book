# Web Request

In the `main` function, we will call our function to initialize Wi-Fi. Once it is initialized, we will call `send_http_request`, which we will define shortly.

```rust
let wifi_stack = access_website::wifi::init_wifi(spawner, peripherals.WIFI).await;

info!("Wi-Fi Initialized");
Timer::after(Duration::from_millis(1000)).await;

send_http_request(wifi_stack).await;
```

## Web Request and Response

We will work on the `send_http_request` function.

```rust
async fn send_http_request(wifi_stack: embassy_net::Stack<'static>) {
    // ...
}
```

First, we create the TCP and DNS clients, which we then use to create the HTTP client.

```rust
let tcp_client = TcpClient::new(
    wifi_stack,
    mk_static!(
        TcpClientState
        <1, 1500, 1500>,
        TcpClientState::<1, 1500, 1500>::new()
    ),
);
let dns_client = DnsSocket::new(wifi_stack);

let mut client = HttpClient::new(&tcp_client, &dns_client);
```

We will send the web request to the `httpbin.org` website. You can use any other website and try sending a request. To receive the response, we create a buffer. We then send the request using the buffer to store the response.

```rust
let mut rx_buf = [0u8; 4096];

let mut builder = client
    .request(Method::GET, "http://httpbin.org/get?hello=Hello+esp-hal")
    .await
    .inspect_err(|e| error!("Request Build Error: {:?}", e))
    .unwrap();

// let mut builder = builder.headers(&[("Host", "httpbin.org"), ("Connection", "close")]);

info!("Sending HTTP Request");
let response = builder.send(&mut rx_buf).await.unwrap();
```

We then read the response body and print it. 

```rust
match response.body().read_to_end().await {
    Ok(data) => {
        if let Ok(st) = core::str::from_utf8(data) {
            info!("Body: {}", st);
        }
    }
    Err(e) => info!("Body error: {:?}", e),
}
```

We can further process the response body as JSON using crates like `serde`, but we will stop here and cover this in later chapters.

## Run the Program

Unlike the previous examples, we need to pass the Wi-Fi credentials as environment variables. You can either set them in your shell beforehand or prefix the command with the variables like this:

```
SSID='YOUR_WIFI_NAME' PASSWORD='YOUR_WIFI_PASSWORD' cargo run --release
```

## Clone the existing project

You can clone the project I created or refer to the existing project and navigate to the `access-website` folder.

```sh
git clone https://github.com/ImplFerris/esp32c5-projects
cd esp32c5-projects/access-website/
```
