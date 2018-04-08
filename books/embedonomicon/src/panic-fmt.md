# `panic_fmt`

`panic_fmt` is a language item that defines the behavior of `panic!`. This lang item is a function
and at the time of writing it has the following signature:

``` rust
#[lang = "panic_fmt"]
unsafe extern "C" fn panic_fmt(
    _args: core::fmt::Arguments,
    _file: &'static str,
    _line: u32,
    _col: u32,
) -> ! {
    // ..
}
```

You should always check what the exact signature of `panic_fmt` is; using the wrong signature will
produce nasty bugs (like increased binary size) or undefined behavior (when it's not defined as a
divergent function). You can find the exact signature of `panic_fmt` in `src/libcore/panicking.rs`.

Even if your program never `panic!`s the compiler will still ask you to define this lang item.
After you add that lang item (you can stub it with an infinite `loop {}` for now) you'll get a new
error:

``` rust
#![feature(lang_items)]
#![no_std]

fn main() {}

#[lang = "panic_fmt"]
unsafe extern "C" fn panic_fmt(
    _args: core::fmt::Arguments,
    _file: &'static str,
    _line: u32,
    _col: u32,
) -> ! {
    loop {}
}
```

``` console
$ xargo build --target thumbv7m-none-eabi
error: requires `start` lang_item
```
