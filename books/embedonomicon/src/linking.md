# Linking

If you try to link the previous program you'll get an error:

``` console
$ xargo build --target thumbv7m-none-eabi
error: linking with `arm-none-eabi-gcc` failed: exit code: 1
  |
  = note: "arm-none-eabi-gcc" "-L" (..)
  = note: /usr/arm-none-eabi/lib/crt0.o: In function `_start':
          (.text+0x90): undefined reference to `memset'
          (.text+0xd0): undefined reference to `atexit'
          (.text+0xd4): undefined reference to `__libc_init_array'
          (.text+0xe4): undefined reference to `exit'
          (.text+0x100): undefined reference to `__libc_fini_array'
          collect2: error: ld returned 1 exit status

error: aborting due to previous error
```

Here you can see that `rustc` is using `gcc` to link the program. `gcc` by default will try to link
in a bunch of C stuff to make the program more POSIX-y but we don't need all that. If we switch the
linker to `ld` we'll be free of the C stuff so let's do that:

``` console
$ xargo rustc --target thumbv7m-none-eabi -- \
    -C linker=arm-none-eabi-ld -Z linker-flavor=ld
```

Now the program links! But the output binary is actually empty ...

``` console
$ arm-none-eabi-size target/thumbv7m-none-eabi/debug/app
   text    data     bss     dec     hex filename
      0       0       0       0       0 target/thumbv7m-none-eabi/debug/app

$ arm-none-eabi-objdump -Cd target/thumbv7m-none-eabi/debug/app

target/thumbv7m-none-eabi/debug/app:     file format elf32-littlearm
```

Turns out the linker discarded everything because it didn't find an *entry point*.

We can tell the linker what we want the entry point to be by passing the `-e` flag to it. For now
let's use `main` (the one produced by `rustc`) as the entry point.

``` console
$ xargo rustc --target thumbv7m-none-eabi -- \
    -C linker=arm-none-eabi-ld -Z linker-flavor=ld -C link-arg=-emain
```

Now the output binary is not empty!

``` console
$ arm-none-eabi-size target/thumbv7m-none-eabi/debug/app
   text    data     bss     dec     hex filename
    162       0       0     162      a2 target/thumbv7m-none-eabi/debug/app
```

But this program is wrong in many ways ... ARM Cortex-M devices expect firmware to have a certain
memory layout and this program doesn't have it.

``` console
$ arm-none-eabi-objdump -Cd target/thumbv7m-none-eabi/debug/app
```

``` armasm
target/thumbv7m-none-eabi/debug/app:     file format elf32-littlearm


Disassembly of section .text:

00008000 <app::main>:
    8000:       4770            bx      lr

00008002 <app::start>:
    8002:       b580            push    {r7, lr}
    (..)
    8030:       bd80            pop     {r7, pc}

00008032 <<() as app::Termination>::report>:
    8032:       b081            sub     sp, #4
    8034:       2000            movs    r0, #0
    8036:       b001            add     sp, #4
    8038:       4770            bx      lr

0000803a <main>:
    803a:       b5d0            push    {r4, r6, r7, lr}
    (..)
    8076:       bdd0            pop     {r4, r6, r7, pc}
```

We can fix that using a linker script.
