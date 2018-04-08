# Compiler intrinsics

If you compile a program like this:

``` rust
#![no_std]

extern crate glue;

use core::ptr;

fn main() {
    let a = unsafe { ptr::read_volatile(0x2000_0000 as *const u64) };
    let b = unsafe { ptr::read_volatile(0x2000_0008 as *const u64) };

    let c = a * b;
    unsafe { ptr::write_volatile(0x2000_0010 as *mut u64, c) }
}
```

You'll run into a linker error:

``` console
error: linking with `arm-none-eabi-ld` failed: exit code: 1
  |
  = note: "arm-none-eabi-ld" "-L" (..)
  = note: $/thumbv7m-none-eabi/debug/deps/app-$.$.rcgu.o: In function `app::main':
          /home/japaric/tmp/app/src/main.rs:11: undefined reference to `__aeabi_uldivmod'


error: aborting due to previous error
```

The issue is that the ARM Cortex-M architecture doesn't natively support multiplication of 64-bit
integers as in there's no hardware instruction to perform that operation. So the compiler, actually
LLVM, lowers the multiplication to a call to a compiler intrinsic; in this case it lowers the
multiplication to a call to the `__aeabi_uldivmod` intrinsic.

The `core` crate doesn't provide this, or any other, *compiler intrinsic* for complicated reasons
but you can get the intrinsics by linking to the `compiler_builtins` crate. This crate contains a
Rust implementation of all compiler intrinsics that LLVM uses.

To link to the `compiler_builtins` crate first you need to instruct Xargo to build it:

``` toml
# Xargo.toml
[dependencies.core]
stage = 0

[dependencies.compiler_builtins]
features = ["mem"]
stage = 1
```

Then you simply add `extern crate compiler_builtins` somewhere in your dependency graph:

``` rust
#![feature(compiler_builtins_lib)] // NEW!
#![no_std]

extern crate compiler_builtins; // NEW!
extern crate glue;

use core::ptr;

fn main() {
    let a = unsafe { ptr::read_volatile(0x2000_0000 as *const u64) };
    let b = unsafe { ptr::read_volatile(0x2000_0008 as *const u64) };

    let c = a * b;
    unsafe { ptr::write_volatile(0x2000_0010 as *mut u64, c) }
}
```
