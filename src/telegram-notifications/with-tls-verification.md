# TLS Verification

In the previous version, we disabled TLS certificate verification, which made the connection vulnerable to MITM attacks. In this chapter, we will work on the same project but properly verify the server certificate. You can update your existing project for the relevant parts.

To verify the server certificate, we could embed the `api.telegram.org` certificate itself, but that would tightly couple our firmware to the current server certificate. Telegram can replace its certificate when it expires or when its certificate configuration changes. Instead, we store the root CA certificate. The TLS library uses this root certificate to verify the server certificate.

> [!Tip]
> If you get stuck or run into any import errors, you can refer to my project and navigate to the `telegram-tls` folder
>
> [ESP32-C5 Projects](https://github.com/ImplFerris/esp32c5-projects/tree/main/telegram-tls)

## Check the Certificate Chain

To download the correct root certificate, we first need to find out which CA issued the certificate used by `api.telegram.org`. You can skip this step also. If you are curious, you can run the following command to see which certificate authority issued the certificate for the Telegram API domain.

```sh
echo | openssl s_client \
    -connect api.telegram.org:443 \
    -servername api.telegram.org 2>/dev/null |
    grep -E " s:| i:"
```

If you run the command, you should see that the certificate chain uses GoDaddy certificates.

## Download the Root CA Certificate

Since the certificate authority is GoDaddy, we can download the GoDaddy G2 root CA certificate from its official certificate repository.

```sh
mkdir -p certs

curl -o certs/root.pem \
    https://certs.godaddy.com/repository/gdroot-g2.crt

# Verify the certificate
openssl x509 \
    -in certs/root.pem \
    -noout \
    -subject \
    -enddate
```

`reqwless` expects the CA certificate in DER format, so convert the downloaded PEM certificate to DER:

```sh
openssl x509 \
    -in certs/root.pem \
    -outform DER \
    -out certs/root.der
```

Store the certificate in a `certs` directory at the project root:

```sh
.
├── build.rs
├── Cargo.toml
├── certs
│   └── root.der
├── rust-toolchain.toml
├── src
│   ├── bin
│   │   └── main.rs
│   ├── lib.rs
│   ├── telegram.rs
│   └── wifi.rs
```

## Configure Certificate Verification

Now we can include the DER certificate in the firmware and use it to verify the server certificate. Replace `TlsVerify::None` with the `TlsVerify::Certificate` block:

```rust
const ROOT_CERT: &[u8] = include_bytes!("../certs/root.der");

let tls_config = TlsConfig::new(
    tls_seed,
    &mut self.tls_rx,
    &mut self.tls_tx,
    TlsVerify::Certificate {
        // Verifying the certificate
        ca: ROOT_CERT,
        cert: None,
        key: None,
    },
);
```

The ca field contains the root CA certificate that we trust. The TLS library uses it to verify the server certificate.

## Clone the existing project

You can clone the project I created or refer to the existing project and navigate to the `telegram-tls` folder.

```sh
git clone https://github.com/ImplFerris/esp32c5-projects
cd esp32c5-projects/telegram-tls/
```
