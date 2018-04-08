# `glue` crate

Seems like a good time to move all the unstable bits out of the user program. After all we don't
want the end user to deal with those. We'll be moving them into a `glue` crate.

``` console
$ cargo new --lib glue

$ mv src/main.rs glue/src/lib.rs

$ edit glue/src/lib.rs && cat $_
```

``` rust
#![feature(lang_items)]
#![no_std]

#[lang = "panic_fmt"]
unsafe extern "C" fn panic_fmt(
    _args: core::fmt::Arguments,
    _file: &'static str,
    _line: u32,
    _col: u32,
) -> ! {
    loop {}
}

#[lang = "start"]
extern "C" fn start<T>(user_main: fn() -> T, _argc: isize, _argv: *const *const u8) -> isize
where
    T: Termination,
{
    user_main().report() as isize
}

#[lang = "termination"]
trait Termination {
    fn report(self) -> i32;
}

impl Termination for () {
    fn report(self) -> i32 {
        0
    }
}
```

``` console
$ edit src/main.rs && cat $_
```

``` rust
#![no_std]

extern crate glue;

fn main() {}
```

``` console
$ edit Cargo.toml && tail -n2 $_
```

``` toml
[dependencies]
glue = { path = "glue" }
```

``` console
$ xargo build

$ arm-none-eabi-size target/thumbv7m-none-eabi/debug/app
   text    data     bss     dec     hex filename
      0       0       0       0       0 target/thumbv7m-none-eabi/debug/app

$ xargo rustc -- -C link-arg=-emain

$ arm-none-eabi-size target/thumbv7m-none-eabi/debug/app
   text    data     bss     dec     hex filename
    162       0       0     162      a2 target/thumbv7m-none-eabi/debug/app
```
