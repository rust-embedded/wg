# `PROVIDE`

Another way to implement weak aliasing is to use the `PROVIDE` linker script feature. Using this
feature the relevant code in the `glue` crate would look like this:

``` rust
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
            #[used]
            static KEEP: extern "C" fn() = $exception;

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

// the global_asm! call is gone

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

// ..
```

And the linker script would look like this:

``` text
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}

ENTRY(reset);

EXTERN(default_handler);

/* NEW! */
PROVIDE(nmi = default_handler);
PROVIDE(hard_fault = default_handler);
PROVIDE(memory_management_fault = default_handler);
PROVIDE(bus_fault = default_handler);
PROVIDE(usage_fault = default_handler);
PROVIDE(svcall = default_handler);
PROVIDE(pendsv = default_handler);
PROVIDE(systick = default_handler);

SECTIONS
{
  .vector_table :
  {
    LONG(ORIGIN(RAM) + LENGTH(RAM)); /* Initial Stack Pointer value */
    KEEP(*(.vector_table.reset_vector));
    KEEP(*(.vector_table.exceptions));
  } > FLASH

  /* .. */
}
```

The user interface remains the same.
