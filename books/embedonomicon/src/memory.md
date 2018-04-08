# `MEMORY`

Microcontrollers have at least two different types of memory: some form of non volatile memory,
usually Flash, and Random Access Memory (RAM).

The first thing that a linker script must specify is how the different parts of a program are
located in these (different) memory regions.

A minimal linker script that contains information about memory regions would look like this:

``` console
$ edit link.x && cat $_
```

``` text
/* Memory layout of the STM32F103C8T6 */
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM   : ORIGIN = 0x20000000, LENGTH = 20K
}

SECTIONS
{
  .text :             /* the output .text section ... */
  {
    *(.text .text.*); /* contains .text and .text.* section from ALL input files and ... */
  } > FLASH           /* is placed in the FLASH memory region */
}
```

The `MEMORY` part of this linker script describes two memory regions: one named `FLASH` that starts
at address `0x0800_0000` and has a size of 64 KiB (1 KiB = 1024 bytes), and another named `RAM` that
starts at address `0x2000_0000` and has a size of 20 KiB.

The `SECTIONS` part says that the `.text` section of the *output* binary will be placed in the
`FLASH` memory region. The `*(.text .text.*)` bit means "place the `.text` and `.text.*` (`*` is a
glob) sections from all (`*(..);`) the input files in this section".

To make the linker use this linker script we have to pass the flag `-Tlink.x` to it and the easiest
way is to set that flag in the Cargo configuration file.

``` console
$ edit .cargo/config && cat $_
```

``` toml
[target.thumbv7m-none-eabi]
rustflags = [
  "-C", "link-arg=-Tlink.x", # <- NEW!
  "-C", "linker=arm-none-eabi-ld",
  "-Z", "linker-flavor=ld",
]

[build]
target = "thumbv7m-none-eabi"
```

``` console
$ xargo rustc -- -C link-arg=-emain

$ arm-none-eabi-objdump -Cd target/thumbv7m-none-eabi/debug/app
```

``` armasm
target/thumbv7m-none-eabi/debug/app:     file format elf32-littlearm

Disassembly of section .text:

08000000 <glue::start>:
 8000000:       b580            push    {r7, lr}
 (..)
 800002e:       bd80            pop     {r7, pc}

08000030 <app::main>:
 8000030:       4770            bx      lr

08000032 <main>:
 8000032:       b5d0            push    {r4, r6, r7, lr}
 (..)
 800006e:       bdd0            pop     {r4, r6, r7, pc}

08000070 <<() as glue::Termination>::report>:
 8000070:       b081            sub     sp, #4
 8000072:       2000            movs    r0, #0
 8000074:       b001            add     sp, #4
 8000076:       4770            bx      lr
```

Note how the addresses of the subroutines have changed; they are now all in the `FLASH` memory
region. The compiler will place all subroutines in the `.text` section of an object file.

`.text` is not the only special section. `.rodata` is the section where all constants (like strings)
will be placed. `.bss` contains all the `static` variables whose initial value is zero, and `.data`
contains all the `static` variables whose initial value is not zero.

``` rust
#![no_std]

extern crate glue;

use core::ptr;

// [0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x2c, 0x20, 0x77, 0x6f, 0x72, 0x6c, 0x64, 0x21]
static RODATA: &str = "Hello, world!";

static mut BSS: u32 = 0;
static mut DATA: u32 = 1;

fn main() {
    // force the compiler the keep these symbols in the final program
    unsafe {
        ptr::read_volatile(RODATA.as_bytes().as_ptr());
        ptr::read_volatile(&BSS);
        ptr::read_volatile(&DATA);
    }
}
```

If you compile this and inspect the output binary using `objdump` you'll see all those sections I
mentioned above:

``` console
$ arm-none-eabi-objdump -CD target/thumbv7m-none-eabi/debug/app
```

``` armasm
(..)

Disassembly of section .data._ZN3app4DATA17h2bfb531d3ecc03daE:

08000134 <app::DATA>:
 8000134:	00000001    andeq	r0, r0, r1

(..)

Disassembly of section .rodata..Lbyte_str.0:

08000164 <.rodata..Lbyte_str.0>:
 8000164:	6c6c6548    cfstr64vs	mvdx6, [ip], #-288	; 0xfffffee0
 8000168:	77202c6f    strvc	r2, [r0, -pc, ror #24]!
 800016c:	646c726f    strbtvs	r7, [ip], #-623	; 0xfffffd91
 8000170:	Address 0x0000000008000170 is out of bounds.


Disassembly of section .rodata._ZN3app6RODATA17h54b845fa673d1275E:

08000174 <app::RODATA>:
 8000174:	08000164    stmdaeq	r0, {r2, r5, r6, r8}
 8000178:	0000000d    andeq	r0, r0, sp

(..)

Disassembly of section .bss._ZN3app3BSS17h07c344444fdcc7d4E:

0800017c <app::BSS>:
 800017c:	00000000    andeq	r0, r0, r0
```

The problem is that with the current linker script all these `static` variables are placed in the
wrong memory region.

``` console
$ arm-none-eabi-nm -C target/thumbv7m-none-eabi/debug/app
(..)
080000d8 T main
08000140 V __rustc_debug_gdb_scripts_section__
0800017c b app::BSS
08000134 d app::DATA
0800007c t app::main
08000174 r app::RODATA
08000116 T <str as core::str::StrExt>::as_bytes
0800012c T <() as glue::Termination>::report
08000044 T core::ptr::read_volatile
08000062 T core::ptr::read_volatile
08000000 T glue::start
08000030 T <[T] as core::slice::SliceExt>::as_ptr
```

We'll have to update the linker script to place these variables in `RAM` .

``` text
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}

SECTIONS
{
  .text :
  {
    *(.text .text.*);
  } > FLASH

  .rodata :
  {
    *(.rodata .rodata.*);
  } > FLASH

  .data :
  {
    *(.data .data.*);
  } > RAM

  .bss :
  {
    *(.bss .bss.*);
  } > RAM

  /DISCARD/ :
  {
    *(.ARM.exidx.*)
  }
}
```

While we are at it we can drop the `.ARM.exidx` section. This section stores information related to
unwinding but since we are not to make use of that we can drop it to save some bytes.

Now the `static` variables are stored in the right place.

``` console
$ arm-none-eabi-objdump -CD target/thumbv7m-none-eabi/debug/app
```

``` armasm
(..)

Disassembly of section .rodata:

08000134 <app::RODATA-0x10>:
 8000134:	6c6c6548    cfstr64vs	mvdx6, [ip], #-288	; 0xfffffee0
 8000138:	77202c6f    strvc	r2, [r0, -pc, ror #24]!
 800013c:	646c726f    strbtvs	r7, [ip], #-623	; 0xfffffd91
 8000140:	00000021    andeq	r0, r0, r1, lsr #32

08000144 <app::RODATA>:
 8000144:	08000134    stmdaeq	r0, {r2, r4, r5, r8}
 8000148:	0000000d    andeq	r0, r0, sp

(..)

Disassembly of section .data:

20000000 <app::DATA>:
20000000:	00000001    andeq	r0, r0, r1

Disassembly of section .bss:

20000004 <app::BSS>:
20000004:	00000000    andeq	r0, r0, r0
```

``` console
$ arm-none-eabi-nm -C target/thumbv7m-none-eabi/debug/app
(..)
080000d8 T main
08000154 V __rustc_debug_gdb_scripts_section__
20000004 b app::BSS
20000000 d app::DATA
0800007c t app::main
08000144 r app::RODATA
08000116 T <str as core::str::StrExt>::as_bytes
0800012c T <() as glue::Termination>::report
08000044 T core::ptr::read_volatile
08000062 T core::ptr::read_volatile
08000000 T glue::start
08000030 T <[T] as core::slice::SliceExt>::as_ptr
```

This new linker script also makes the output of `size -Ax` more readable:

``` console
$ arm-none-eabi-size -Ax target/thumbv7m-none-eabi/debug/app
target/thumbv7m-none-eabi/debug/app  :
section                size         addr
.text                 0x134    0x8000000
.rodata                0x18    0x8000134
.debug_gdb_scripts     0x22    0x800014c
.data                   0x4   0x20000000
.bss                    0x4   0x20000004
```
