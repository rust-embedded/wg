# `compiler_builtins`

`compiler_builtins` provides compiler (LLVM) intrinsics implemented in Rust but it's unstable to
use. A workaround for this unstable feature is to use the `libgcc.a` library; this library provides
a C implementation of the compiler intrinsics.

Unfortunately, linking to `libgcc.a` requires switching to `arm-none-eabi-gcc` as a linker. The
required changes are as follows:

`Xargo.toml` is no longer required:

``` console
$ rm Xargo.toml
```

The linker needs to be changed to `arm-none-eabi-gcc` in the Cargo configuration file:

``` console
$ cat .cargo/config
```

``` toml
[target.thumbv7m-none-eabi]
rustflags = [
  "-C", "link-arg=-Tlink.x",
  "-C", "link-arg=-nostartfiles",
]

[build]
target = "thumbv7m-none-eabi"
```

To link to `libgcc.a` this can be added to the `glue` crate:

``` rust
#![feature(lang_items)]
#![feature(linkage)]
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

#[link(name = "gcc")] // NEW!
extern "C" {
    fn nmi();
    fn hard_fault();
    fn memory_management_fault();
    fn bus_fault();
    fn usage_fault();
    fn svcall();
    fn pendsv();
    fn systick();
}

// ..
```

With these changes the following program will continue to compile:

``` rust
#![no_std]

#[macro_use(exception)]
extern crate glue;

use core::ptr;

fn main() {
    let a = unsafe { ptr::read_volatile(0x2000_0000 as *const u64) };
    let b = unsafe { ptr::read_volatile(0x2000_0008 as *const u64) };

    let c = a * b;
    unsafe { ptr::write_volatile(0x2000_0010 as *mut u64, c) }
}

exception!(default_handler, dh);

fn dh() {
    loop {}
}
```

# `__aeabi_mem*`

`libgcc.a` doesn't provide the `__aebi_mem*` symbols so you'll run into linking errors if you try to
compile a program like this:

``` rust
#![no_std]

#[macro_use(exception)]
extern crate glue;

use core::ptr;

fn main() {
    let xs = [0; 128];

    unsafe {
        ptr::read_volatile(&xs[0]);
    }
}

exception!(default_handler, dh);

fn dh() {
    loop {}
}
```

`libc.a` provides the `__aeabi_mem*` symbols so you can link to that library to fix this problem:

- `glue/src/lib.rs`

``` rust
// ..

#[link(name = "c")] // <- NEW!
#[link(name = "gcc")]
extern "C" {
    fn nmi();
    fn hard_fault();
    fn memory_management_fault();
    fn bus_fault();
    fn usage_fault();
    fn svcall();
    fn pendsv();
    fn systick();
}

// ..
```
