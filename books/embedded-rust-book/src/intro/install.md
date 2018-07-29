> **⚠️: This section contains exports from [Japaric's Discovery] book.**
>
> Contents should be reviewed for consistency in the context
> of this book before "publishing"

[Japaric's Discovery]: https://japaric.github.io/discovery/

> **⚠️: This section still references `nightly` Rust**
>
> Contents should be updated to work on `stable` Rust when possible

> **⚠️: This section has not been checked as of 2018-07-29**
>
> Contents should be checked to still be working with current `nightly`
> or `stable` Rust

# Setting up a development environment

Dealing with microcontrollers involves several tools as we'll be dealing with an architecture
different than your laptop's and we'll have to run and debug programs on a "remote" device.

## Documentation

Tooling is not everything though. Without documentation is pretty much impossible to work with
microcontrollers.

We'll be referring to all these documents throughout this book:

*HEADS UP* All these links point to PDF files and some of them are hundreds of pages long and
several MBs in size.

- [STM32F3DISCOVERY User Manual][um]
- [STM32F303VC Datasheet][ds]
- [STM32F303VC Reference Manual][rm]
- [LSM303DLHC]
- [L3GD20]

[L3GD20]: http://www.st.com/resource/en/datasheet/l3gd20.pdf
[LSM303DLHC]: http://www.st.com/resource/en/datasheet/lsm303dlhc.pdf
[ds]: http://www.st.com/resource/en/datasheet/stm32f303vc.pdf
[rm]: http://www.st.com/resource/en/reference_manual/dm00043574.pdf
[um]: http://www.st.com/resource/en/user_manual/dm00063382.pdf

## Tools

We'll use all the tools listed below. Where a minimum version is not specified, any recent version should work but we have listed the version we have tested.

- Cargo & `rustc` >= nightly-2018-06-04
- [`itmdump`] v0.2.1
- OpenOCD >=0.8. Tested versions: v0.9.0 and v0.10.0
- `arm-none-eabi-gcc`. Tested version: ?.??
- `arm-none-eabi-binutils`
    - `arm-none-eabi-ld`. Tested version: 2.30
    - `arm-none-eabi-gdb`. Version 7.12 or newer highly recommended. Tested versions: 7.10, 7.11,
      7.12 and 8.1
- `minicom` on Linux and macOS. Tested version: 2.7. Readers report that `picocom` also works but
  we'll use `minicom` in this text.

- `PuTTY` on Windows.

[`itmdump`]: https://crates.io/crates/itm

If your laptop has Bluetooth functionality and you have the Bluetooth module, you can additionally
install these tools to play with the Bluetooth module. All these are optional:

- Linux, only if you don't have a Bluetooth manager application like Blueman.
  - `bluez`
  - `hcitool`
  - `rfcomm`
  - `rfkill`

macOS / OSX / Windows users only need the default bluetooth manager that ships with their OS.

Next, follow OS-agnostic installation instructions for a few of the tools:

### `rustc` & Cargo

Install rustup by following the instructions at [https://rustup.rs](https://rustup.rs).

Then, install or switch to the nightly channel.

``` console
$ rustup default nightly
```

**NOTE** Make sure you have a nightly newer than `nightly-2018-06-04`. `rustc -V` should return a
date newer than the one shown below:

``` console
$ rustc -V
rustc 1.28.0-nightly (29f48ccf3 2018-06-03)
```

### `itmdump`

``` console
$ cargo install itm --vers 0.2.1
```

### OS specific instructions

Now follow the instructions specific to the OS you are using:

- [Linux](intro/install/linux.html)
- [Windows](intro/install/windows.html)
- [macOS](intro/install/macos.html)
