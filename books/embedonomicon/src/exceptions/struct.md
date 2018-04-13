# `Exceptions` struct

The simplest approach is to have the user install all the exception vectors in their program. This
approach requires tweaking the linker script as follows:

``` text
SECTIONS
{
  .vector_table :
  {
    LONG(ORIGIN(RAM) + LENGTH(RAM));
    KEEP(*(.vector_table.reset_vector));
    KEEP(*(.vector_table.exceptions));
  } > FLASH

  /* .. */
}
```

And then providing some types in the `glue` crate to make this less error prone.

``` console
$ edit glue/src/lib.rs && cat $_
```

``` rust
#![feature(lang_items)]
#![feature(used)]
#![no_std]

use core::ptr;

/// Exception vectors
// The 14 entries in the vector table that follow the reset vector
#[repr(C)]
pub struct Exceptions {
    // 0x08
    pub nmi: extern "C" fn(),
    // 0x0C
    pub hard_fault: extern "C" fn(),
    // 0x10
    pub memory_management_fault: extern "C" fn(),
    // 0x14
    pub bus_fault: extern "C" fn(),
    // 0x18
    pub usage_fault: extern "C" fn(),
    pub reserved4: [Reserved; 4],
    // 0x2C
    pub svcall: extern "C" fn(),
    pub reserved2: [Reserved; 2],
    // 0x3C
    pub pendsv: extern "C" fn(),
    // 0x40
    pub systick: extern "C" fn(),
}

/// Reserved exception vector
// newtype used to ensure that zero is always written to the reserved slots
#[derive(Clone, Copy)]
pub struct Reserved(usize);

// Default exception handler
extern "C" fn default_handler() {
    loop {}
}

/// Exception vectors that all point to the default exception handler
pub const DEFAULT_EXCEPTION_HANDLERS: Exceptions = Exceptions {
    nmi: default_handler,
    hard_fault: default_handler,
    memory_management_fault: default_handler,
    bus_fault: default_handler,
    usage_fault: default_handler,
    reserved4: [Reserved(0); 4],
    svcall: default_handler,
    reserved2: [Reserved(0); 2],
    pendsv: default_handler,
    systick: default_handler,
};

/* .. */
```

`#[repr(C)]` is crucial to get the exception vectors in the memory layout specified in the
documentation.

Then the user can insert the exception vectors in the vector table doing the following:

``` rust
#![feature(asm)]
#![feature(used)]
#![no_std]

extern crate glue;

use core::ptr;

use glue::Exceptions;

fn main() {
    unsafe {
        // invalid operation that raises a Hard Fault exception
        ptr::read_volatile(0x2100_0000 as *const u32);
    }
}

extern "C" fn my_hard_fault_handler() {
    unsafe { asm!("bkpt" :::: "volatile") }
}

#[link_section = ".vector_table.exceptions"]
#[used]
static EXCEPTIONS: Exceptions = Exceptions {
    hard_fault: my_hard_fault_handler,  // override
    ..glue::DEFAULT_EXCEPTION_HANDLERS
};
```

This is the generated machine code:

``` armasm
target/thumbv7m-none-eabi/debug/app:     file format elf32-littlearm

Disassembly of section .vector_table:

08000000 <glue::RESET_VECTOR-0x4>:
 8000000:	20005000

08000004 <glue::RESET_VECTOR>:
 8000004:	080000e7

08000008 <app::EXCEPTIONS>:
 8000008:	080000e3
 800000c:	080000a1  ; <-
 8000010:	080000e3
 8000014:	080000e3
 8000018:	080000e3
    ...
 800002c:	080000e3
    ...
 8000038:	080000e3
 800003c:	080000e3

Disassembly of section .text:

08000040 <glue::start>:
 (..)

08000070 <core::ptr::read_volatile>:
 (..)

0800008a <app::main>:
 (..)

080000a0 <app::my_hard_fault_handler>:  ; <-
 (..)

080000a4 <main>:
 (..)

080000e2 <glue::default_handler>:  ; *
 (..)

080000e6 <reset>:
 (..)

08000104 <<() as glue::Termination>::report>:
 (..)

0800010c <core::ptr::null>:
 (..)
```

There are a few problems with this implementation so we'll refine it.

First, the user can forget add `#[link_section]`, `#[used]` or the whole `EXCEPTIONS` variable.
There's also nothing stopping the user from assigning a type different than `Exceptions` to the
`EXCEPTIONS` variable. In all of these cases the problem described [here] will occur.

[here]: ../exceptions.html

The easiest way to fix most of the problems is to abstract away the `static` variable using a macro.

``` rust
// in glue/src/lib.rs

#[macro_export]
macro_rules! exceptions {
    ($($exception:ident: $handler:ident),+,) => {
        #[link_section = ".vector_table.exceptions"]
        #[used]
        static EXCEPTIONS: $crate::Exceptions = $crate::Exceptions {
            $($exception: $handler,)+
            ..$crate::DEFAULT_EXCEPTION_HANDLERS
        };
    }
}
```

Now the user doesn't have to deal with the `static` variable.

``` rust
#![feature(asm)]
#![feature(used)]
#![no_std]

#[macro_use(exceptions)]
extern crate glue;

use core::ptr;

fn main() {
    unsafe {
        ptr::read_volatile(0x2100_0000 as *const u32);
    }
}

extern "C" fn my_hard_fault_handler() {
    unsafe { asm!("bkpt" :::: "volatile") }
}

exceptions! {
    hard_fault: my_hard_fault_handler,
}
```

This produces the exact same machine code as before.

The `exceptions!` macro solves most of the problems, except the one where the user forgets to call
the `exceptions!` macro. In that scenario the exception vectors won't be added to the vector table.
What we can do in that case is raise an error at linking time using an assertion in the linker
script:

``` text
SECTIONS
{
  .vector_table :
  {
    LONG(ORIGIN(RAM) + LENGTH(RAM));
    KEEP(*(.vector_table.reset_vector));
    KEEP(*(.vector_table.exceptions));
    _evector = .;
  } > FLASH

  /* .. */
}

ASSERT(_evector == ORIGIN(FLASH) + 0x40, "you forgot to call `exceptions!`");
```

Now if the user forgets to call the `exceptions!` they'll get the following error:

``` console
$ cargo build
error: linking with `arm-none-eabi-ld` failed: exit code: 1
  |
  = note: "arm-none-eabi-ld" "-L" (..)
  = note: arm-none-eabi-ld: you forgot to call `exceptions!`

error: aborting due to previous error
```
