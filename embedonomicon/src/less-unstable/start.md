# `start` / `termination`

To provide a standard `main` interface to user we have to use two unstable features: the `start`
lang item (function) and the `termination` lang item (trait). We can remove the need for both by
using a more raw symbol interface to glue the user `main` to the entry point (`reset`) in glue:

- `glue/src/lib.rs`

``` rust
#![feature(lang_items)] // still needed for `panic_fmt`
#![no_std]

#[no_mangle]
pub unsafe extern "C" fn reset() -> ! {
    extern "C" {
        fn main() -> !; // <- CHANGED! We can now pick any signature we want

        static mut _sbss: u32;
        static mut _ebss: u32;

        static mut _sdata: u32;
        static mut _edata: u32;
        static _sidata: u32;
    }

    zero_bss(&mut _sbss, &mut _ebss);
    initialize_data(&mut _sdata, &mut _edata, &_sidata);

    main()
}

// ..

// #[lang = "start"] and #[lang = "termination"] have been dropped
```

But now the user needs to tweak their program a bit:

``` rust
#![no_main] // <- NEW!
#![no_std]

#[macro_use(exception)]
extern crate glue;

// CHANGED!
#[no_mangle]
pub extern "C" fn main() -> ! {
    loop {}
}

exception!(default_handler, dh);

fn dh() {
    loop {}
}
```

We have seen this unmangled symbol interface and we know it's type safe and easy to get wrong. But
as before we can add type safety by wrapping it in a macro:

- `glue/src/lib.rs`

``` rust
#![feature(lang_items)]
#![no_std]

#[macro_export]
macro_rules! main {
    ($path:path) => {
        #[export_name = "main"]
        #[no_mangle]
        pub extern "C" fn __impl_main() -> ! {
            let f: fn() -> ! = $path;
            f()
        }
    }
}

// ..
```

Now the user can write:

``` rust
#![no_main] // <- still needed
#![no_std]

#[macro_use(exception, main)]
extern crate glue;

main!(main);

fn main() -> ! {
    loop {}
}

exception!(default_handler, dh);

fn dh() {
    loop {}
}
```

Which is less error prone as the user main is type checked to have the correct signature.
