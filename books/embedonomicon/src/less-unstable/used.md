# `#[used]`

In general `#[used]` can be replaced by using `EXTERN` in the linker script. The two uses of
`#[used]` in the `glue` crate are `RESET_VECTOR` and `EXCEPTIONS`; those two can be dropped by
making the following modifications:

- `glue/src/lib.rs`

``` rust
#![feature(lang_items)]
#![feature(linkage)]
// #![feature(used)] // no longer needed
#![no_std]

// ..

#[link_section = ".vector_table.exceptions"]
#[no_mangle] // NEW! Also this static is now `pub`lic
pub static EXCEPTIONS: [Option<unsafe extern "C" fn()>; 14] = [
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

#[link_section = ".vector_table.reset_vector"]
#[no_mangle] // NEW! Also this static is now `pub`lic
pub static RESET_VECTOR: unsafe extern "C" fn() -> ! = reset;
```

- `link.x`

``` text
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}

ENTRY(reset);

EXTERN(default_handler);
EXTERN(RESET_VECTOR); // NEW!
EXTERN(EXCEPTIONS); // NEW!

/* .. */
```

It should be noted that the downside of using `EXTERN` is that the `EXTERN`-ed symbol will *always*
be kept in the final binary, regardless of whether the program refers to the symbol or not. This is
not really a problem in the case of `RESET_VECTOR` and `EXCEPTIONS` because those must be present in
the final binary for it to operate correctly.
