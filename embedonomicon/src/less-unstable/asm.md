# `asm!`

Sometimes you may need to use assembly in your application but the inline assembly macro, `asm!`, is
an unstable feature. The stable workaround is to compile an external assembly file and call the
assembly subroutine using FFI. Example below:

- `asm.s`

``` armasm
.global syscall

syscall:
  bkpt 0xAB
  bx lr
```

- `build.rs`

``` rust
extern crate cc;

fn main() {
    cc::Build::new().file("asm.s").compile("asm");
}
```

- `src/main.rs`

``` rust
#![no_std]

#[macro_use(exception)]
extern crate glue;

// The important part
extern "C" {
    fn syscall(nr: usize, arg: usize) -> usize;
}

// Prints "Hello, world" to the host console using semihosting
fn main() {
    const OPEN: usize = 0x01;
    const WRITE: usize = 0x05;

    const W_TRUNC: usize = 4;

    const TT: &[u8] = b":tt\0";

    let args = [TT.as_ptr() as usize, W_TRUNC, TT.len()];
    let fd = unsafe { syscall(OPEN, args.as_ptr() as usize) };

    if fd as isize == -1 {
        // error
    } else {
        const MSG: &[u8] = b"Hello, world!\n";

        let args = [fd, MSG.as_ptr() as usize, MSG.len()];
        unsafe { syscall(WRITE, args.as_ptr() as usize) };
    }
}

exception!(default_handler, dh);

fn dh() {
    loop {}
}
```

The downside of this approach is that calling the assembly subroutine always incurs in an overhead
of a function call, that is the assembly subroutine won't be inlined. This is not only a performance
issue; the function call overhead makes it impossible to use this approach for operations that need
to inspect the current stack frame like reading the Program Counter or the Link Register.
