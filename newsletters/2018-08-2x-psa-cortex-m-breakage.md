> PSA: building binaries and cdylibs for the ARM Cortex-M architecture will break with today's nightly

The default linker for the 4 ARM Cortex-M targets listed below has changed from
`arm-none-eabi-gcc` to `rust-lld` in the latest nightly.

- `thumbv6m-none-eabi`
- `thumbv7m-none-eabi`
- `thumbv7em-none-eabi`
- `thumbv7em-none-eabihf`

This will break the builds of *binaries* and *cdylibs* that were using the
old default linker (`arm-none-eabi-gcc`) *and* additionally pass extra flags to
the linker using any of these rustc flags: `-C link-arg`, `-C link-args`, `-Z
pre-link-arg` or `-Z pre-link-args`. Building libraries (`rlib`s and
`staticlib`s) is not affected by this change.

This change won't affect stable users when it reaches the 1.30 release because,
as of 1.28, it's not possible to build binaries or cdylibs for those targets on
the stable channel. Building libraries for those targets is possible on stable
but it's not affected by this change.

### Rationale

This breaking change was intentional.

We, the [embedded WG], wanted to reduce the number of external tools required to
build embedded programs for the ARM Cortex-M architecture. By switching the
default linker to the LLD linker that's shipped with the Rust toolchain the user
no longer needs to install an ARM linker (like `arm-none-eabi-gcc` or
`arm-none-eabi-ld`) to build Rust programs.

[embedded WG]: https://github.com/rust-embedded/wg

Before landing this change we first [consulted] with the community if they
thought this breaking change was worth it. We received over 20 positive responses
representing the Cortex-M team (part of the embedded WG), the Tock OS project,
the embed-rs organization and independent developers.

The consensus was that it was worth to make the default configuration more self
contained and that if we were to make the change it had to be made before it
became possible to build binaries on stable otherwise it wouldn't be possible
to make this change without breaking stable builds.

[consulted]: https://github.com/rust-embedded/wg/issues/160

### How to fix your build

If you are affected by this change you'll observe a linker error with a message
similar to one shown below:

``` console
$ # these are the custom linker flags of the project
$ cat .cargo/config
```

``` toml
[target.thumbv7m-none-eabi]
runner = 'arm-none-eabi-gdb'
rustflags = [
  "-C", "link-arg=-Wl,-Tlink.x",
  "-C", "link-arg=-nostartfiles",
]

[build]
target = "thumbv7m-none-eabi"
```

```
$ cargo build
error: linking with `rust-lld` failed: exit code: 1
  |
  = note: "rust-lld" "-flavor" "gnu" (..)
  = note: rust-lld: error: unknown argument: -Wl,-Tlink.x
          rust-lld: error: unknown argument: -nostartfiles
```

There are two ways to fix the problem.

#### Option A: switch back to GCC

The easiest way is to switch back to using `arm-none-eabi-gcc` as the linker. To
do so pass the flag `-C linker=arm-none-eabi-gcc` to rustc. In the above example
you can do that in the `.cargo/config` file.

``` console
$ # these are the custom linker flags of the project
$ cat .cargo/config
```

``` toml
[target.thumbv7m-none-eabi]
runner = 'arm-none-eabi-gdb'
rustflags = [
  "-C", "linker=arm-none-eabi-gcc", # ADDED
  "-C", "link-arg=-Wl,-Tlink.x",
  "-C", "link-arg=-nostartfiles",
]

[build]
target = "thumbv7m-none-eabi"
```

```
$ cargo build && echo It works now
It works now
```

#### Option B: tweak the additional linker arguments

The other option is to tweak the additional linker arguments so they'll be
accepted by LLD. In the above example the `-nostartfiles` flag can be dropped
because that's the default behavior of LLD, and the flags prefixed by `-Wl,`
will have to lose their prefix.

``` console
$ # these are the custom linker flags of the project
$ cat .cargo/config
```

``` toml
[target.thumbv7m-none-eabi]
runner = 'arm-none-eabi-gdb'
rustflags = [
  "-C", "link-arg=-Tlink.x", # CHANGED
# "-C", "link-arg=-nostartfiles", # REMOVED
]

[build]
target = "thumbv7m-none-eabi"
```

``` console
$ cargo build && echo It works now
It works now
```

With this approach your build will no longer depend on an external linker.

#### Should I prefer option A or B?

If you are linking to a system installed C library like `newlib-arm-none-eabi`
then you should continue to use GCC. The default library search path of
`arm-none-eabi-gcc` includes the path to those libraries.

If you are not linking to any C code then you should prefer LLD then you won't
need to install the `arm-none-eabi` toolchain.
