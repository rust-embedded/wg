# Modular linker scripts

So far we have used a *monolithic* linker script but we can split it in several linker scripts that
cover architecture agnostic details, architecture specific details and device specific details.

- Architecture agnostic linker script

``` text
/* glue/link.x */

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
    *(.data .data.*);
  } > RAM

  .bss :
  {
    *(.bss .bss.*);
  } > RAM
}
```

- ARM Cortex-M specific linker script

``` text
/* glue/cortex-m.x */

ENTRY(reset);

EXTERN(default_handler);
EXTERN(RESET_VECTOR);
EXTERN(EXCEPTIONS);

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
  .vector_table ORIGIN(FLASH) :
  {
    LONG(ORIGIN(RAM) + LENGTH(RAM)); /* Initial Stack Pointer value */
    KEEP(*(.vector_table.reset_vector));
    KEEP(*(.vector_table.exceptions));
  } > FLASH

  /DISCARD/ :
  {
    *(.ARM.exidx.*)
  }
}
```

- Device specific linker script

``` text
/* memory.x */
MEMORY
{
  FLASH : ORIGIN = 0x08000000, LENGTH = 64K
  RAM : ORIGIN = 0x20000000, LENGTH = 20K
}
```

Only with these changes the application will fail to link with:

``` console
error: linking with `arm-none-eabi-gcc` failed: exit code: 1
  |
  = note: "arm-none-eabi-gcc" "-L" (..)
  = note: /usr/arm-none-eabi/bin/ld: cannot open linker script file link.x: No such file or directory
          collect2: error: ld returned 1 exit status

error: aborting due to previous error
```

Because the linker looks for linker scripts only in the current directory and the library search
path. To fix this we have to make the `glue` crate put the linker scripts in the library search
path using a build script:

- `glue/build.rs`

``` rust
use std::env;
use std::fs::File;
use std::io::Write;
use std::path::PathBuf;

fn main() {
    // Put the linker scripts somewhere the linker can find it
    let out = &PathBuf::from(env::var_os("OUT_DIR").unwrap());

    File::create(out.join("link.x"))
        .unwrap()
        .write_all(include_bytes!("link.x"))
        .unwrap();

    File::create(out.join("cortex-m.x"))
        .unwrap()
        .write_all(include_bytes!("cortex-m.x"))
        .unwrap();

    println!("cargo:rustc-link-search={}", out.display());
}
```
