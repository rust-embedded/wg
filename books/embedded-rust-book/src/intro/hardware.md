> **⚠️: This section contains exports from [Japaric's Discovery] book.**
>
> Contents should be reviewed for consistency in the context
> of this book before "publishing"

[Japaric's Discovery]: https://japaric.github.io/discovery/

# Meet your hardware

Let's get familiar with the hardware we'll be working with.

## STM32F3DISCOVERY (the "F3")

<p align="center">
<img title="F3" src="assets/f3.jpg">
</p>

We'll refer to this board as "F3" throughout this book.

What does this board contain?

- A STM32F303VCT6 microcontroller. This microcontroller has
  - A single core ARM Cortex-M4F processor with hardware support for single precision floating point
    operations and a maximum clock frequency of 72 MHz.

  - 256 KiB of "Flash" memory. (1 KiB = 10**24** bytes)

  - 48 KiB of RAM.

  - many "peripherals": timers, GPIO, I2C, SPI, USART, etc.

  - lots of "pins" that are exposed in the two lateral "headers".

  - **IMPORTANT** This microcontroller operates at (around) 3.3V.

- An [accelerometer] and a [magnetometer][] (in a single package).

[accelerometer]: https://en.wikipedia.org/wiki/Accelerometer
[magnetometer]: https://en.wikipedia.org/wiki/Magnetometer

- A [gyroscope].

[gyroscope]: https://en.wikipedia.org/wiki/Gyroscope

- 8 user LEDs arranged in the shape of a compass

- A second microcontroller: a STM32F103CBT. This microcontroller is actually part of an on-board
  programmer and debugger named ST-LINK and is connected to the USB port named "USB ST-LINK".

- There's a second USB port, labeled "USB USER" that is connected to the main microcontroller, the
  STM32F303VCT6, and can be used in applications.

