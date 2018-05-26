> **⚠️: This section contains exports from [Japaric's Discovery] book.**
>
> Contents should be reviewed for consistency in the context
> of this book before "publishing"

[Japaric's Discovery]: https://japaric.github.io/discovery/

> **⚠️: This section has not been checked as of 2018-05-26**
>
> Contents should be checked to still be working with current `nightly`
> or `stable` Rust

# Windows

## `arm-none-eabi-*`

ARM provides `.exe` installers for Windows. Grab one from [here][gcc], and follow the instructions.
Just before the installation process finishes tick/select the "Add path to environment variable"
option. Then verify that the tools are in your `%PATH%`:

``` console
$ arm-none-eabi-gcc -v
(..)
gcc version 5.4.1 20160919 (release) (..)
```

[gcc]: https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads

## OpenOCD

There's no official binary release of OpenOCD for Windows but there are unofficial releases
available [here][openocd]. Grab the 0.10.x zipfile and extract it somewhere in your drive (I
recommend `C:\OpenOCD` but with the drive letter that makes sense to you) then update your `%PATH%`
environment variable to include the following path: `C:\OpenOCD\bin` (or the path that you used
before).

[openocd]: https://github.com/gnu-mcu-eclipse/openocd/releases

Verify that OpenOCD is in yout `%PATH%` with:

``` console
$ openocd -v
Open On-Chip Debugger 0.10.0
(..)
```

## PuTTY

Download the latest `putty.exe` from [this site] and place it somewhere in your `%PATH%`.

[this site]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html

## ST-LINK USB driver

You'll also need to install [this USB driver] or OpenOCD won't work. Follow the installer
instructions and make sure you install the right (32-bit or 64-bit) version of the driver.

[this USB driver]: http://www.st.com/en/embedded-software/stsw-link009.html

That's all! Go to the [next section].

[next section]: intro/install/verify.html
