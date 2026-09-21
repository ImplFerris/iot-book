# Embedded Basics

Embedded systems combine hardware and software to perform a specific task. Unlike typical computers, they are designed for a specific purpose and often have limited resources. Before we get started, let's refresh some of the basics.

## MCU vs Module vs Development Board

A microcontroller (MCU) is a small computer built into a single chip. It contains a processor, memory, and peripherals that allow it to interact with the outside world.

The ESP32-C5 is a microcontroller from Espressif. It integrates a RISC-V processor, memory, peripherals, and wireless connectivity into a single chip. 

You might have come across the term "module". A module is basically the microcontroller with the components needed to use it, such as flash memory and an antenna. The ESP32-C5-WROOM-1 is an example of an ESP32-C5 module.

A development board provides an easy way to work with the module by adding components such as USB connectivity, power circuitry, and accessible pins. The one we are using for this book, the Waveshare ESP32-C5-WIFI6-KIT, is a development board.

## Peripherals

Peripherals are hardware components inside the microcontroller that provide specific functions. The ESP32-C5 includes peripherals such as GPIO, UART, I2C, SPI, ADC, and timers.

### GPIO

GPIO stands for General Purpose Input/Output. GPIO pins can be configured as inputs or outputs to interact with external hardware. For example, a GPIO can be used to control an LED or detect a button press.
