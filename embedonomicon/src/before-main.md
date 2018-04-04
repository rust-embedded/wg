# Life before `main`

Our program can properly boot and we can do many things in `main` but something is still broken: all
`static` variables are uninitialized -- that is they have random contents -- by the time we reach
`main` and that results in undefined behavior if the program relies on the initial value of a
`static` variable.

It's standard in C program to have all statically allocated variables initialized before reaching
the user `main`. In our Rust program before main corresponds to the reset handler so let's
initialize the `static` variables there.

We'll need help from the linker script to do the initialization:

- `glue/link.x`

``` text
INCLUDE memory.x;
INCLUDE cortex-m.x;

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
    /* NEW! */
    _sidata = LOADADDR(.data);
    . = ALIGN(4);
    _sdata = .;

    *(.data .data.*);

    /* NEW! */
    . = ALIGN(4);
    _edata = .;
  } > RAM AT > FLASH /* <- CHANGED! */

  .bss :
  {
    /* NEW! */
    . = ALIGN(4);
    _sbss = .;

    *(.bss .bss.*);

    /* NEW! */
    . = ALIGN(4);
    _ebss = .;
  } > RAM
}
```

This basically gives names to the start and end addresses of the `.bss` and `.data` sections. The `.
= ALIGN(4)` part ensures that these addresses are 4-byte aligned. We'll use this alignment to
optimize the initialization routine.

The initial values of the variables in the `.data` section are stored in `FLASH` (this is what the
`AT > FLASH` means in the linker script); `LOADADDR(.data)` returns the `FLASH` address to those
initial values.

Finally the initialization in `glue` looks like this:

- `glue/src/lib.rs`

``` rust
// Reset handler
#[no_mangle]
pub unsafe extern "C" fn reset() -> ! {
    extern "C" {
        fn main(argc: isize, argv: *const *const u8) -> isize;

        static mut _sbss: u32;
        static mut _ebss: u32;

        static mut _sdata: u32;
        static mut _edata: u32;
        static _sidata: u32;
    }

    zero_bss(&mut _sbss, &mut _ebss);
    initialize_data(&mut _sdata, &mut _edata, &_sidata);

    main(0, ptr::null());

    loop {}
}

unsafe fn zero_bss(sbss: *mut u32, ebss: *mut u32) {
    let mut bss = sbss;
    while bss < ebss {
        // NOTE(ptr::write*) to force aligned stores
        // NOTE(volatile) to prevent the compiler from optimizing this into `memclr`
        ptr::write_volatile(bss, 0);
        bss = bss.offset(1);
    }
}

unsafe fn initialize_data(sdata: *mut u32, edata: *mut u32, sidata: *const u32) {
    let mut data = sdata;
    let mut idata = sidata;
    while data < edata {
        // NOTE(ptr::{read,write}) to force aligned loads and stores
        ptr::write(data, ptr::read(idata));
        data = data.offset(1);
        idata = idata.offset(1);
    }
}
```

Now this program will execute without panicking.

``` rust
#![no_std]

#[macro_use(exception)]
extern crate glue;

use core::ptr;

static mut BSS: u16 = 0;
static mut DATA: u16 = 1;

fn main() {
    unsafe {
        assert_eq!(ptr::read_volatile(&BSS), 0);
        assert_eq!(ptr::read_volatile(&DATA), 1);
    }
}

exception!(default_handler, dh);

fn dh() {
    loop {}
}
```

You'll also note that the `.bss` and `.data` sections are now 4-byte padded:

``` console
$ arm-none-eabi-size target/thumbv7m-none-eabi/release/app
   text    data     bss     dec     hex filename
    174       4       4     182      b6 target/thumbv7m-none-eabi/release/app
```
