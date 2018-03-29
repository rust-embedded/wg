# Introduction

Welcome to "The Embedded Rust Book", an introductory book about using the Rust Programming Language on "Bare Metal" embedded systems, such as Microcontrollers.

## Who Embedded Rust is For

`TODO`

## Who This Book is For

This book assumes the following:

* You are comfortable using the Rust Programming Language, and have written, run, and debugged Rust applications on a desktop environment
* You are comfortable developing and debugging embedded systems in another language such as C, C++, or ADA, and are familiar with concepts such as:
    * Cross Compilation
    * Linker Scripts
    * Memory Mapped Peripherals
    * Interrupts
    * Common interfaces such as I2C, SPI, Serial, and others

If you are not yet comfortable with Rust, we highly suggest completing the [Rust Book] before attempting to learn with this book.

If you are not yet comfortable with Embedded Systems, we highly suggest `LEARNING WITH OTHER RESOURCES` before attempting to learn with this book.

[Rust Book]: https://doc.rust-lang.org/book/second-edition

## How to Use This Book

This book generally assumes that you’re reading it front-to-back, that is, later chapters build on top of concepts in earlier chapters, and earlier chapters may not dig into details on a topic, revisiting the topic in a later chapter.

This book will be using the `UNDEFINED` development board from `UNDEFINED` for the majority of the examples contained within. This board is based on the ARM Cortex-M architecture, and while basic functionality is common across most CPUs based on this architecture, peripherals and other implementation details of Microcontrollers are different between different vendors, and often even different between Microcontroller families from the same vendor.

For this reason, we suggest purchasing the `UNDEFINED` development board for the purpose of following this book.

## Contributing to This Book

Until the initial release of this book (planned to coincide with the 2018 Era release of the Rust Programming Language), the work on this book will be coordinated in [this tracking issue] of the [Embedded Working Group] repository.

Pull requests are very welcome.

[this tracking issue]: https://github.com/rust-lang-nursery/embedded-wg/issues/56
[Embedded Working Group]: https://github.com/rust-lang-nursery/embedded-wg