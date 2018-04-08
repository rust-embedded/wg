# `start`

Basically to be able to use the `main` function as normal Rust programs do you need to define the
`start` lang item. The signature of the `start` function at the time of writing is the following:

``` rust
#[lang = "start"]
extern "C" fn start<T>(_main: fn() -> T, _argc: isize, _argv: *const *const u8) -> isize
where
    T: Termination,
{
    // ..
}
```

## `termination`

Currently to define the `start` function you also need to define the `Termination` trait using the
`termination` lang item as follows:

``` rust
#[lang = "termination"]
trait Termination {
    fn report(self) -> i32;
}
```

At the very least you'll have to implement this trait for the unit type `()` otherwise `fn main() {
/* .. */ }` won't be accepted in user programs.

## How does it work?

Suppose the user wrote a binary crate named `app` that contains a `main` function. When compiling
this crate the compiler will look for the `start` lang item in the dependency graph of the `app`
crate. Let's say it finds it in module `m` of the dependency `d`. The compiler will proceed to
append an unmangled `main` function that calls the `start` lang item . In terms of Rust code it
would look like this:

``` rust
/* start of user code */

extern crate d;

// ..

fn main() {
    // ..
}

/* end of user code */

// inserted by the compiler
#[export_name = "main"]
pub extern "C" fn rustc_main(argc: isize, argv: *const *const u8) -> isize {
    d::m::start(main, argc, argv)
}
```

---

## Symbols

Putting all together you'll end with something like this

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

Let's first turn the crate into an object file using:

``` console
$ # NOTE disable incremental compilation to reproduce these steps
$ xargo rustc --target thumbv7m-none-eabi -- --emit=obj -C linker=true # don't link

$ find -name '*.o'
./target/thumbv7m-none-eabi/debug/deps/app-6e6a2da6f83262fd.o
```

Now we can look at the symbols in this object file using `nm`:

``` console
$ arm-none-eabi-nm -C ./target/thumbv7m-none-eabi/debug/deps/app-6e6a2da6f83262fd.o
(..)
00000000 T main
00000000 V __rustc_debug_gdb_scripts_section__
00000000 t app::main
00000000 T app::start
00000000 t <() as app::Termination>::report
```

`main` is the unmangled symbol inserted by the compiler; `app::main` is the empty user main; and
`app::start` is the `start` lang item.
