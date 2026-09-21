{{#title Embedded Rust Ecosystem}}

# Embedded Rust Ecosystem

Traditionally, languages such as C and C++ have been widely used for embedded development. Rust is becoming another option for embedded development. I believe Rust is a good fit for building secure IoT devices.

Let's look at some of the main components of the ecosystem that we'll use in this book.

## `no_std`

When you write a typical Rust application, it uses the `std` crate, which provides functionality such as collections, file system access, networking, and threads. But many of these features depend on an operating system. Our microcontroller does not have an operating system running underneath our application. Instead, our program runs directly on the microcontroller.

For this reason, embedded Rust applications commonly use `#![no_std]`, which tells Rust not to use the `std` crate. Instead, we can use the `core` crate, which provides fundamental Rust functionality without requiring an operating system.

## PAC and HAL

PAC stands for Peripheral Access Crate. It gives us direct access to the microcontroller's peripherals and registers. PACs are usually generated from SVD files that describe the microcontroller's hardware.

HAL stands for Hardware Abstraction Layer. It works on top of the PAC and provides a higher-level interface for working with the microcontroller hardware without having to access hardware registers directly.

Most of the time, we will be working with the HAL directly. However, sometimes the HAL does not provide access to a particular feature or register we need. In that case, we can use the PAC directly.

## esp-hal

We will be using [`esp-hal`](https://github.com/esp-rs/esp-hal/) throughout this book. `esp-hal` is a `no_std` Rust HAL for Espressif's ESP32 chips. With `esp-hal`, we can access the underlying PAC when needed.

The `esp-hal` repository also contains basic examples that you can refer to when needed.

## Rust `std` on ESP32

Another option for Rust development on ESP32 chips is to use Rust's standard library (`std`) together with ESP-IDF.

[ESP-IDF](https://github.com/espressif/esp-idf) is Espressif's framework, primarily written in C, for developing applications for ESP32 chips. The [Rust on ESP-IDF](https://github.com/esp-rs/esp-idf) repository contains Rust crates that build on top of ESP-IDF.

I usually prefer working with `no_std` because it makes it easier to move embedded Rust code between different microcontrollers. Many microcontrollers' HALs do not provide `std`.

## embedded-hal

`embedded-hal` is not a HAL like `esp-hal`. It is a foundation for building an ecosystem of platform-agnostic drivers. It provides common interfaces that drivers can use to communicate with external devices, such as sensors or displays.

For example, a sensor driver can use the `embedded-hal` I2C interface instead of depending on a specific microcontroller or HAL. This allows the same driver to work with different microcontrollers and HALs that implement the required traits.

## Embassy

Embassy is an asynchronous framework for embedded Rust. It provides an async executor that allows us to run multiple tasks. When a task is waiting, the executor can run another task that is ready to make progress.

