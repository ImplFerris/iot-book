# Web Server on ESP32-C5

In this section, let's learn how to create a web server and run it on the ESP32-C5.

> Heads up: don't have high hopes when I say "web server". We are not going to expose it to the internet. That would be a fun little exercise, but in my opinion, it is not very practical to run a website from an MCU.

Instead, we will run a web server that can be accessed only within our local network. For example, you can connect your phone or laptop to the same Wi-Fi network as the ESP32-C5 and open the ESP32-C5's IP address in a browser.

**Then, what is the purpose of a web server on the ESP32-C5?**

Imagine you have an ESP32-C5 connected to a light, motor, or some sensors. You could run a small web interface on the ESP32-C5 and use your phone or laptop to monitor and control the device through a browser. 

Also, instead of serving the web interface from the device, you could keep the ESP32-C5 lightweight and expose an HTTP API while a separate mobile or desktop app provides the user interface. Both approaches can work entirely within your local network without relying on a cloud service.

> [!Tip]
> If you get stuck or run into any import errors, you can refer to my project and navigate to the `web-server` folder
>
> [ESP32-C5 Projects](https://github.com/ImplFerris/esp32c5-projects/tree/main/web-server)


## Project Setup

Run the following command to create the `web-server` project:

```sh
esp-generate --headless \
  -o esp32c5 \
  -o esp32c5-wroom-1-psram \
  -o unstable-hal \
  -o alloc \
  -o embassy \
  -o wifi \
  -o defmt \
  web-server
```

Once project is created, copy the Wi-Fi module along with the `mk_static` macro and Wi-Fi initialization code from the previous chapters.

## Picoserve

The main crate we are going to use is `picoserve`. It is an async `no_std` HTTP server suitable for bare-metal environments, heavily inspired by Axum. We will use it to run our web server.

Add the following to `Cargo.toml`:

```toml
picoserve = { version = "0.20.1", features = ["embassy"] }
```

The picoserve repository has many nice examples to get started. You can check them out [here](https://github.com/sammhicks/picoserve/tree/development/examples).


## Assets

We will use a simple HTML page with a Ferris image on it. You can create pretty much any web page you like and use it instead. If you want to use mine, download the following two files and put them in a new `assets` directory in your project root.

- [index.html](https://raw.githubusercontent.com/ImplFerris/esp32c5-projects/refs/heads/main/web-server/assets/index.html) 
- [logo.svg](https://raw.githubusercontent.com/ImplFerris/esp32c5-projects/refs/heads/main/web-server/assets/logo.svg)

## Project Structure

Our project structure will look like this:

```sh
.
├── assets
│   ├── index.html
│   └── logo.svg
├── build.rs
├── Cargo.toml
├── rust-toolchain.toml
├── src
│   ├── bin
│   │   └── main.rs
│   ├── lib.rs
│   ├── server.rs
│   └── wifi.rs
```

I have also created a `server` module where we will handle the routing and run the web server.

## Server Configuration

First, import the picoserve types we will use to create the router and serve our HTML files.

```rust
use picoserve::{Router, response::File, routing};

pub const WEB_TASK_POOL_SIZE: usize = 2;
static CONFIG: picoserve::Config = picoserve::Config::const_default().keep_connection_alive();
```

`WEB_TASK_POOL_SIZE` controls how many web server tasks we can run concurrently; two is enough for us. `CONFIG` is the picoserve configuration. We just set everything to the default and keep HTTP connections alive so the client can reuse the same connection for multiple requests.

## Web Server Task

Now we can create the web server task. The task receives the network stack and a `task_id`, then creates the buffers needed by picoserve.

As you can see, `WEB_TASK_POOL_SIZE` specifies how many instances of `web_task` can run concurrently. We need multiple instances so picoserve can handle multiple HTTP connections concurrently. We use the same value when spawning the tasks, so we create exactly as many tasks as the pool allows.

```rust
#[embassy_executor::task(pool_size = WEB_TASK_POOL_SIZE)]
pub async fn web_task(task_id: usize, stack: embassy_net::Stack<'static>) -> ! {
    let port = 80;
    let mut tcp_rx = [0; 1024];
    let mut tcp_tx = [0; 1024];
    let mut http_buffer = [0; 2048];

    let app = Router::new()
        .route(
            "/",
            routing::get_service(File::html(include_str!("../assets/index.html"))),
        )
        .route(
            "/logo.svg",
            routing::get_service(File::with_content_type(
                "image/svg+xml",
                include_bytes!("../assets/logo.svg"),
            )),
        );
        
    picoserve::Server::new(&app, &CONFIG, &mut http_buffer)
        .listen_and_serve(task_id, stack, port, &mut tcp_rx, &mut tcp_tx)
        .await
        .into_never()
}
```

The `/` route serves our `index.html` file, while `/logo.svg` serves the SVG image. If you are serving CSS or JavaScript, you can also use `File::css` or `File::javascript`. For SVG or other images, we can use `with_content_type` instead to set the content type to `image/svg+xml`.

Finally, `listen_and_serve` starts the server on port `80` and waits for incoming HTTP connections.

## Start Web Server Tasks

From `main`, we spawn the web server tasks using the same `WEB_TASK_POOL_SIZE` we defined in `server.rs`.

```rust
for task_id in 0..server::WEB_TASK_POOL_SIZE {
    spawner.spawn(server::web_task(task_id, wifi_stack).unwrap());
}

info!("Started web tasks");
```

The loop gives each task a unique `task_id`, which picoserve uses to identify the server task. With `WEB_TASK_POOL_SIZE` set to `2`, this starts two web server tasks that can handle connections concurrently.

## Run the Program

Now let's run the program and connect the ESP32-C5 to your Wi-Fi network:

```
SSID='YOUR_WIFI_NAME' PASSWORD='YOUR_WIFI_PASSWORD' cargo run --release
```

You should see output similar to this, including the IP address:

```text
[INFO ] Got IP: 192.168.0.101/24 ...
...
[INFO ] Started web tasks ...
```

Once the ESP32-C5 has connected to the network and received an IP address, open that IP address in a browser from another device (mobile or laptop) connected to the same Wi-Fi network.

```text
http://192.168.0.101
```
