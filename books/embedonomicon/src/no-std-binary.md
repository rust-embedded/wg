# A no-std binary

What's the simplest way to put together a `no-std` binary?

Your first try will probably be something like this:

``` console
$ cargo new --bin app && cd $_

$ edit src/main.rs && cat $_
```

``` rust
#![no_std]

fn main() {}
```

``` console
$ cargo build --target thumbv7m-none-eabi
error: language item required, but not found: `panic_fmt`
```
