# Vector table

> TODO mention alias between 0x0000_0000 and 0x0800_0000 on STM32 devices

Our program is looking better: it places instructions and `static` variables in the right memory
region (`FLASH` or `RAM`), but our program won't boot because it's missing the vector table.

According to [ARM documentation][vt] the vector table must be located at address `0x0000_0000`. The
vector table is basically a list (array) of pointers. The first two entries in this table are:

[vt]: https://developer.arm.com/docs/dui0552/latest/the-cortex-m3-processor/exception-model/vector-table

- The initial stack pointer value
- The reset *vector*

The first entry basically defines where the (call) stack will be located. In the ARM architecture
the stack grows downwards so this initial stack pointer value usually points to the end of the RAM
region.

In ARM terminology a *vector* is basically a function pointer that the processor calls on a certain
event, and a *handler* is the function a vector points to. Thus the reset vector points to the reset
handler, a function that's called on a reset condition.

When an ARM Cortex-M device boots it first initializes the stack pointer to the value stored in the
first entry of the vector table and then calls the reset handler -- the second entry of the vector
table points to this function.

Just so you get the idea this how the boot process would look like if it was implemented as Rust
code:

``` rust
static VECTOR_TABLE: [usize; N] = /* .. */;
static mut STACK_POINTER: usize;  // undefined on boot

unsafe fn boot() -> ! {
    STACK_POINTER = VECTOR_TABLE[0];
    mem::transmute::<_, fn() -> !>(VECTOR_TABLE[1])();
}
```

Now the important question is how to encode these constraints in the linker script. This can be
accomplish in two steps.

First we add the reset handler and the reset vector to the `glue` crate.

``` console
$ edit glue/src/lib.rs && head $_
```

``` rust
#![feature(lang_items)]
#![feature(used)]
#![no_std]

use core::ptr;

// Reset handler
#[no_mangle]
pub unsafe extern "C" fn reset() -> ! {
    extern "C" {
        fn main(argc: isize, argv: *const *const u8) -> isize;
    }

    main(0, ptr::null());

    loop {}
}

#[link_section = ".vector_table.reset_vector"]
#[used]
static RESET_VECTOR: unsafe extern "C" fn() -> ! = reset;

/* .. */
```

`#[used]` is required to prevent the compiler from optimizing away `RESET_VECTOR` when compiling
with optimizations enabled.

Then we use the linker script to add a `.vector_table` section to the output binary.

``` text
/* .. */

ENTRY(reset);

SECTIONS
{
  .vector_table :
  {
    LONG(ORIGIN(RAM) + LENGTH(RAM)); /* Initial Stack Pointer value */
    KEEP(*(.vector_table.reset_vector));
  } > FLASH

  /* .. */
}
```

With this in place we now have a bootable binary!

``` armasm
target/thumbv7m-none-eabi/debug/app:     file format elf32-littlearm

Disassembly of section .vector_table:

08000000 <glue::RESET_VECTOR-0x4>:
 8000000:	20005000

08000004 <glue::RESET_VECTOR>:
 8000004:	08000135

Disassembly of section .text:

08000008 <glue::start>:
 (..)

08000038 <<[T] as core::slice::SliceExt>::as_ptr>:
 (..)

0800004c <core::ptr::read_volatile>:
 (..)

0800006a <core::ptr::read_volatile>:
 (..)

08000084 <app::main>:
 (..)

080000e0 <main>:
 (..)

0800011e <<str as core::str::StrExt>::as_bytes>:
 (..)

08000134 <reset>:
 (..)

08000152 <<() as glue::Termination>::report>:
 (..)

0800015a <core::ptr::null>:
 (..)
```

You'll note that the second entry of the vector table is `0x0800_0135`; this value doesn't match the
address of the `reset` handler: the value is off by 1. This is OK because when the least significant
bit of a function pointer is set to 1 it indicates that the pointer points to a function that will
be executed in Thumb mode. In ARMv7-M all functions are executed in Thumb mode so all function
pointers have their least significant bit set to 1.

Our approach holds even in presence of optimizations thanks to the `#[used]` attribute.

``` armasm
target/thumbv7m-none-eabi/release/app:     file format elf32-littlearm

Disassembly of section .vector_table:

08000000 <glue::RESET_VECTOR-0x4>:
 8000000:	20005000

08000004 <glue::RESET_VECTOR>:
 8000004:	0800002b

Disassembly of section .text:

08000008 <main>:
 (..)

0800002a <reset>:
 (..)

08000036 <<() as glue::Termination>::report>:
 (..)
```

And even with LTO:

``` armasm
target/thumbv7m-none-eabi/release/app:     file format elf32-littlearm

Disassembly of section .vector_table:

08000000 <glue::RESET_VECTOR-0x4>:
 8000000:	20005000

08000004 <glue::RESET_VECTOR>:
 8000004:	08000009

Disassembly of section .text:

08000008 <reset>:
 8000008:	f240 0028   movw	r0, #40	; 0x28
 800000c:	f6c0 0000   movt	r0, #2048	; 0x800
 8000010:	7800        ldrb	r0, [r0, #0]
 8000012:	f240 0004   movw	r0, #4
 8000016:	f2c2 0000   movt	r0, #8192	; 0x2000
 800001a:	6800        ldr	r0, [r0, #0]
 800001c:	f240 0000   movw	r0, #0
 8000020:	f2c2 0000   movt	r0, #8192	; 0x2000
 8000024:	6800        ldr	r0, [r0, #0]
 8000026:	e7fe        b.n	8000026 <reset+0x1e>
```
