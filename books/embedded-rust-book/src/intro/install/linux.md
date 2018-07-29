> **⚠️: This section contains exports from [Japaric's Discovery] book.**
>
> Contents should be reviewed for consistency in the context
> of this book before "publishing"

[Japaric's Discovery]: https://japaric.github.io/discovery/

> **⚠️: This section has not been checked as of 2018-07-29**
>
> Contents should be checked to still be working with current `nightly`
> or `stable` Rust

# Linux

Here are the installation commands for a few Linux distributions.

## REQUIRED packages

- Ubuntu 16.04 or newer / Debian Jessie or newer

``` console
$ sudo apt-get install \
  binutils-arm-none-eabi \
  gdb-arm-none-eabi \
  minicom \
  openocd
```

- Fedora 23 or newer

``` console
$ sudo dnf install \
  arm-none-eabi-gcc-cs \
  arm-none-eabi-gdb \
  minicom \
  openocd
```

- Arch Linux

``` console
$ sudo pacman -S \
  arm-none-eabi-gcc \
  arm-none-eabi-gdb \
  arm-none-eabi-gcc \
  minicom

$ yaourt -S openocd
```

## Optional packages

- Ubuntu / Debian

``` console
$ sudo apt-get install \
  bluez \
  rfkill
```

- Fedora

``` console
$ sudo dnf install \
  bluez \
  rfkill
```

- Arch Linux

``` console
$ sudo pacman -S \
  bluez \
  bluez-utils \
  rfkill
```

## udev rules

These rules let you use USB devices like the F3 and the Serial module without root privilege, i.e.
`sudo`.

Create these two files in `/etc/udev/rules.d` with the contents shown below.

``` console
$ cat /etc/udev/rules.d/99-ftdi.rules
```

``` text
# FT232 - USB <-> Serial Converter
ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", GROUP="uucp"
```

``` console
$ cat /etc/udev/rules.d/99-openocd.rules
```

``` text
# STM32F3DISCOVERY rev A/B - ST-LINK/V2
ATTRS{idVendor}=="0483", ATTRS{idProduct}=="3748", GROUP="uucp"

# STM32F3DISCOVERY rev C+ - ST-LINK/V2-1
ATTRS{idVendor}=="0483", ATTRS{idProduct}=="374b", GROUP="uucp"
```

Then reload the udev rules with:

``` console
$ sudo udevadm control --reload-rules
```

If you had any board plugged to your laptop, unplug them and then plug them in again.

Finally, check if you are in the `uucp` group.

``` console
$ groups $(id -nu)
(..) uucp (..)
$ #  ^^^^
```

(`$(id -nu)` returns your user name. In my case it's `japaric`.)

If `uucp` appears in the output. You are all set! Go to the [next section]. Otherwise, keep reading:

[next section]: intro/install/verify.html

- Add yourself to the `uucp` group.

``` console
$ sudo usermod -a -G uucp $(id -u -n)
```

- Check again the output of `groups`. `uucp` should be there this time!

``` console
$ groups $(id -nu)
(..) uucp (..)
$ #  ^^^^
```

You'll have to re-log for these changes to take effect. You have two options:

You can reboot or log out from your current session and then log in; this will close all the
programs you have open right now.

The other option is to use the command below:

``` console
$ su - $(id -nu)
```

to re-log *only in the current shell* and get access to `uucp` devices *only on that shell*. Other
shells *won't* have access to `uucp` devices unless you manually re-log on them with the same `su`
command.

Now, go to the [next section].
