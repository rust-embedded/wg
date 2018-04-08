# Weak aliases

The problem with the `Exceptions` struct approach is that the user has to define *all* the exception
handlers in a single place. This makes it impossible to define exception handlers in different
crates.

Another approach is to use weak aliases. The idea is to populate *all* the exception vectors with
weak functions; the user then is able to individually override each exception handler (because they
are weak functions).

## `glue`

The required changes in the `glue` crate are as follows:

``` rust
#![feature(lang_items)]
#![feature(linkage)]
#![feature(used)]
#![no_std]

use core::ptr;

#[inline(always)]
fn default_handler() -> ! {
    loop {}
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn nmi() {
    default_handler()
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn hard_fault() {
    default_handler()
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn memory_management_fault() {
    default_handler()
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn bus_fault() {
    default_handler()
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn usage_fault() {
    default_handler()
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn svcall() {
    default_handler()
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn pendsv() {
    default_handler()
}

#[linkage = "weak"]
#[no_mangle]
extern "C" fn systick() {
    default_handler()
}

#[link_section = ".vector_table.exceptions"]
#[used]
static EXCEPTIONS: [Option<extern "C" fn()>; 14] = [
    Some(nmi),
    Some(hard_fault),
    Some(memory_management_fault),
    Some(bus_fault),
    Some(usage_fault),
    None, // reserved
    None,
    None,
    None,
    Some(svcall),
    None,
    None,
    Some(pendsv),
    Some(systick),
];

// ..
```

And the linker script needs to be changed as follows:

``` text
/* .. */

SECTIONS
{
  .vector_table :
  {
    LONG(ORIGIN(RAM) + LENGTH(RAM)); /* Initial Stack Pointer value */
    KEEP(*(.vector_table.reset_vector));
    KEEP(*(.vector_table.exceptions)); /* <- NEW! */
  } > FLASH

  /* .. */
}
```

This produces the following machine code:

``` armasm
target/thumbv7m-none-eabi/debug/app:     file format elf32-littlearm

Disassembly of section .vector_table:

08000000 <glue::RESET_VECTOR-0x4>:
 8000000:	20005000

08000004 <glue::RESET_VECTOR>:
 8000004:	080000b1

08000008 <glue::EXCEPTIONS>:
 8000008:	080000cf
 800000c:	080000d3
 8000010:	080000d7
 8000014:	080000db
 8000018:	080000df
    ...
 800002c:	080000e3
    ...
 8000038:	080000e7
 800003c:	080000eb

Disassembly of section .text:

08000040 <glue::start>:
 (..)

08000070 <app::main>:
 8000070:	4770        bx	lr

08000072 <main>:
 (..)

080000b0 <reset>:
 (..)

080000ce <nmi>:
 (..)

080000d2 <hard_fault>:
 (..)

080000d6 <memory_management_fault>:
 (..)

080000da <bus_fault>:
 (..)

080000de <usage_fault>:
 (..)

080000e2 <svcall>:
 (..)

080000e6 <pendsv>:
 (..)

080000ea <systick>:
 (..)

080000ee <<() as glue::Termination>::report>:
 (..)

080000f6 <core::ptr::null>:
 (..)
```

## Overriding an exception

To override one exception the user has to write the following code:

``` rust
#![feature(asm)] // not required for the override functionality
#![no_std]

extern crate glue;

fn main() {}

#[no_mangle]
pub extern "C" fn hard_fault() {
    unsafe { asm!("bkpt" :::: "volatile") }

    loop {}
}
```

This produces the following machine code:

``` armasm
(..)

08000008 <glue::EXCEPTIONS>:
 8000008:       080000b7
 800000c:       08000073
 8000010:       080000bb
 8000014:       080000bf
 8000018:       080000c3
        ...
 800002c:       080000c7
        ...
 8000038:       080000cb
 800003c:       080000cf

(..)

08000072 <hard_fault>:
 8000072:       be00            bkpt    0x0000
 8000074:       e7ff            b.n     8000076 <hard_fault+0x4>
 8000076:       e7fe            b.n     8000076 <hard_fault+0x4>
```


As before we can make this less error prone using a macro:

``` rust
// crate: glue

#![feature(lang_items)]
#![feature(linkage)]
#![feature(used)]
#![no_std]

use core::ptr;

#[macro_export]
macro_rules! exception {
    ($exception:ident, $path:path) => {
        #[no_mangle]
        pub extern "C" fn $exception() {
            let f: fn() = $path; // type check
            f()
        }
    }
}
```

Now the user can write:

``` rust
#![feature(asm)] // not required for the override functionality
#![no_std]

#[macro_use(exception)]
extern crate glue;

fn main() {}

exception!(hard_fault, my_handler);

fn my_handler() {
    unsafe { asm!("bkpt" :::: "volatile") }

    loop {}
}
```

This produces the following machine code:

``` armasm
(..)

08000008 <glue::EXCEPTIONS>:
 8000008:       080000d1
 800000c:       08000079
 8000010:       080000d5
 8000014:       080000d9
 8000018:       080000dd
        ...
 800002c:       080000e1
        ...
 8000038:       080000e5
 800003c:       080000e9

(..)

08000072 <app::my_handler>:
 8000072:       be00            bkpt    0x0000
 8000074:       e7ff            b.n     8000076 <app::my_handler+0x4>
 8000076:       e7fe            b.n     8000076 <app::my_handler+0x4>

08000078 <hard_fault>:
 8000078:       b580            push    {r7, lr}
 800007a:       466f            mov     r7, sp
 800007c:       b082            sub     sp, #8
 800007e:       f240 0073       movw    r0, #115        ; 0x73
 8000082:       f6c0 0000       movt    r0, #2048       ; 0x800
 8000086:       9001            str     r0, [sp, #4]
 8000088:       9801            ldr     r0, [sp, #4]
 800008a:       4780            blx     r0
 800008c:       e7ff            b.n     800008e <hard_fault+0x16>
 800008e:       b002            add     sp, #8
 8000090:       bd80            pop     {r7, pc}
```

## Alias

Our implementation is semantically correct but produces different copies of the default handler: in
the machine code you'll see a `nmi` handler, a `hard_fault` handler, etc. We can fix that using weak
aliases: the core idea is to create a single default exception handler function and have all the
exception vectors point to that single handler, but while retaining the ability to individually
override each exception handler.

Rust the language doesn't support weak aliasing (the C language does) so we have to implement this
using assembly.

``` rust
#![feature(global_asm)]
#![feature(lang_items)]
#![feature(linkage)]
#![feature(used)]
#![no_std]

use core::ptr;

// UNCHANGED
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

#[linkage = "weak"]
#[no_mangle]
pub extern "C" fn default_handler() -> ! {
    loop {}
}

global_asm! {r#"
.weak nmi
nmi = default_handler

.weak hard_fault
hard_fault = default_handler

.weak memory_management_fault
memory_management_fault = default_handler

.weak bus_fault
bus_fault = default_handler

.weak usage_fault
usage_fault = default_handler

.weak svcall
svcall = default_handler

.weak pendsv
pendsv = default_handler

.weak systick
systick = default_handler
"#}

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

#[link_section = ".vector_table.exceptions"]
#[used]
static EXCEPTIONS: [Option<unsafe extern "C" fn()>; 14] = [
    Some(nmi),
    Some(hard_fault),
    Some(memory_management_fault),
    Some(bus_fault),
    Some(usage_fault),
    None,
    None,
    None,
    None,
    Some(svcall),
    None,
    None,
    Some(pendsv),
    Some(systick),
];
```

Nothing changes from the point of view of the user but the machine code is now smaller:

``` rust
target/thumbv7m-none-eabi/debug/app:     file format elf32-littlearm

Disassembly of section .vector_table:

08000000 <glue::RESET_VECTOR-0x4>:
 8000000:	20005000

08000004 <glue::RESET_VECTOR>:
 8000004:	080000d5

08000008 <glue::EXCEPTIONS>:
 8000008:	080000d1
 800000c:	08000079
 8000010:	080000d1
 8000014:	080000d1
 8000018:	080000d1
    ...
 800002c:	080000d1
    ...
 8000038:	080000d1
 800003c:	080000d1

Disassembly of section .text:

08000040 <glue::start>:
 (..)

08000070 <app::main>:
 (..)

08000072 <app::my_handler>:
 (..)

08000078 <hard_fault>:
 (..)

08000092 <main>:
 (..)

080000d0 <bus_fault>:
 (..)

080000d4 <reset>:
 (..)

080000f2 <<() as glue::Termination>::report>:
 (..)

080000fa <core::ptr::null>:
 (..)
```

Note that most of the exception vectors, except for the overridden hard_fault one, now point to the
bus_fault handler.

## Overriding the default handler

I have sneaked in one more feature in the `glue/src/lib.rs` changes: `default_handler` is now a weak
function and thus can be overridden by the user. One more change in the linker script is needed to
make this fully work:

``` text
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}

ENTRY(reset);

EXTERN(default_handler); /* <- NEW! */

SECTIONS
{
  /* .. */
}
```

To override the default handler the procedure is the same as the one we use to override exception
handlers so we can reuse the `exception!` macro:

``` rust
#![feature(asm)] // not required for the override functionality
#![feature(used)]
#![no_std]

#[macro_use(exception)]
extern crate glue;

fn main() {}

exception!(default_handler, my_handler);

fn my_handler() {
    unsafe { asm!("bkpt" :::: "volatile") }

    loop {}
}
```
