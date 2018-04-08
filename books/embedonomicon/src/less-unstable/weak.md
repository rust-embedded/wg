# `#[linkage = "weak"]`

We are using `#[linkage = "weak"]` to make the default exception handler overridable, but we can
drop the use of that feature by having the user *always* provide the default exception handler.

``` rust
#![feature(lang_items)]
// #![feature(linkage)] // no longer needed
#![no_std]

use core::ptr;

#[macro_export]
macro_rules! exception {
    ($exception:ident, $path:path) => {
        #[no_mangle]
        pub extern "C" fn $exception() {
            let f: fn() = $path;
            f()
        }
    }
}

// dropped `default_handler`

// ..
```

The downside is the error message when the user forgets to set the default handler is kind of bad.

``` rust
#![feature(compiler_builtins_lib)]
#![no_std]

extern crate compiler_builtins;
extern crate glue;

fn main() {}
```

``` console
error: linking with `arm-none-eabi-ld` failed: exit code: 1
  |
  = note: "arm-none-eabi-ld" "-L" (..)
  = note: link.x:13: undefined symbol `default_handler' referenced in expression


error: aborting due to previous error
```

The linker error is fixed by defining the default exception handler.

``` rust
#![feature(compiler_builtins_lib)]
#![no_std]

extern crate compiler_builtins;
#[macro_use(exception)]
extern crate glue;

fn main() {}

exception!(default_handler, dh);

fn dh() {
    loop {}
}
```
