- Affected components: `bare-metal`; `cortex-m` and similar
- Feature Name: sound_mutex
- Start Date: 2019-10-23
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

`bare_metal::Mutex`, re-exported in the `cortex-m` and other crates, is a mutex
based on critical sections that temporarily disable all interrupts. As provided
today this abstraction is unsound in multi-core context because `Mutex` can be
stored in `static` variables, which are visible to all cores, but interrupt
masking is *not* sufficient to synchronize (potentially) parallel access (i.e.
access from different cores) to memory.

This document proposes that we deprecate the existing `Mutex` abstraction in
favor of a mutex that properly expresses the idea that mutexes based on
interrupt-masking is only "Sync" in single-core context.

# Motivation

Today it's possible to write multi-core applications for homogeneous multi-core
devices on stable Rust, as evidenced in the [lpcxpresso55S69] repository. In
fact, it has been possible to write multi-core Cortex-M applications since Rust
1.30.0, the release in which it became possible to write single-core Cortex-M
applications on stable.

[lpcxpresso55S69]: https://github.com/japaric/lpcxpresso55S69

Writing homogeneous multi-core applications requires no special build steps and
these programs make use of the same compilation targets used for single-core
applications (e.g. `thumbv7m-none-eabi`). For this reason, it is not possible to
restrict the parts of crates like `cortex-m` that these multi-core applications
can use. Therefore, these crates can *not* assume that they will only be used in
single-core contexts as that may to lead to abstractions that are single-core
sound but multi-core unsound.

`cortex_m::Mutex` is one abstraction that assumes a single-core context. The
following example demonstrates how this "safe" abstraction leads to a data race
in a multi-core application.

``` rust
use core::cell::Cell;
use cortex_m::{Mutex, interrupt};

static X: Mutex<Cell<u64>> = Mutex::new(Cell::new(0));

#[entry(0)] // entry point of core #0
fn main0() {
    interrupt::free(|cs| loop {
        let x = X.borrow(cs);
        x.set(x.get() + 1); // (A)
    });
}

#[entry(1)]
fn main1() {
    interrupt::free(|cs| loop {
        let x = X.borrow(cs);
        x.set(!x.get()); // (B)
    });
}
```

Here the statements A and B run in parallel because disabling the interrupts on
one core has no effect on other cores.

Proliferation of this unsound `Mutex` API, specially in libraries, will
inevitably lead to hard to debug soundness issues in multi-core programs (the
root of the problem could be in a crate deep in the dependency graph). It is
important to promptly remove this unsound API before multi-core applications
become more widespread.

# Detailed design

## `SingleCore*`

This document proposes adopting a new set of `Send` / `Sync` traits that
are "only needs to be sound in single-core context" variants of the ones
provided by the `core` crate. The `Send` and `Sync` traits in `core` will
continue to mean "must be sound in single-core AND multi-core context". These
traits will go into a separate crate so they can be used by other libraries and
frameworks. The full implementation of these traits is shown below -- all names
are up for debate:

``` rust
// crate: single-core-send-sync

pub unsafe auto trait SingleCoreSend {}

pub unsafe auto trait SingleCoreSync {}

// replicate all the `Send` / `Sync` impls in core
impl<T> !SingleCoreSync for *mut T {}
impl<T> !SingleCoreSync for *const T {}
unsafe impl<'a, T> SingleCoreSend for &'a T where T: SingleCoreSync {}
unsafe impl<'a, T> SingleCoreSend for &'a mut T where T: SingleCoreSync {}
// ..

// all multi-core Send types are also single-core Send
unsafe impl<T> SingleCoreSend for T where T: Send {}

// all multi-core Sync types are also single-core Sync
unsafe impl<T> SingleCoreSync for T where T: Sync {}
```

With these traits in place the existing `Mutex` can be replaced with one that
implements `SingleCoreSync`, but not `Sync`.

``` rust
// crate: bare-metal

pub struct SingleCoreMutex<T> { /* .. */ }

unsafe impl<T> SingleCoreSync for SingleCoreMutex<T> {}

impl<T> SingleCoreMutex<T> {
    pub const fn new(data: T) -> Self {
        // ..
    }

    // .. and the rest of the `Mutex` API (e.g. `borrow`) ..
}
```

### Usage

It is not possible to instantiate the `SingleCoreMutex` type in a static
variable because it does not implement the `Sync` trait -- this is a soundness
requirement and not a shortcoming; but existing and upcoming concurrency
frameworks / libraries can loosen their `Send` bounds into `SingleCoreSend`
bounds to accept `SingleCoreMutex` values.

The code snippet below shows an hypothetical API to dynamically register
interrupt handlers. This API uses the `SingleCoreSend` trait instead of the
`Send` bound because it doesn't enable multi-core concurrency (the registered
interrupt handlers will run on core that registered the handler).

``` rust
/// registers an interrupt handler in the vector table of the _calling_ core
fn register(interrupt: Interrupt, handler: F)
where
    F: FnMut() + SingleCoreSend + 'static,
    //           ^^^^^^^^^^^^^^
{
    // ..
}

#[entry]
fn main() {
    // remember: there's a source code level transformation here; `X` and `Y`
    // have type `&'static mut _`
    static mut X: Mutex<Cell<u64>> = Mutex::new(Cell::new(0u64));
    static mut Y: AtomicU32 = AtomicU32::new(0);

    let x: &'static _ = X; // coerces `&'static mut T` into `&'static T`

    register(Interrupt::A, || {
        let z: &'static Mutex<Cell<u64>> = x; // `x` is copied here
        //     ^^^^^^^^^^^^^^^^^^^^^^^^^ OK; this is a `SingleCoreSend` type

        // .. do stuff with `z` ..
    });

    let y: &'static _ = Y;

    register(Interrupt::B, || {
        let z: &'static AtomicU32 = y; // `y` is copied here
        //     ^^^^^^^^^^^^^^^^^^ OK; this is a `Send` type

        // .. do stuff with `z` ..
    });

    loop {
        // do stuff with `x` and `y`
    }
}
```

Here the statically allocated variables, `X` and `Y`, are accessed concurrently
from `main` and the interrupt handlers `A` and `B`. All accesses are performed
by the same core so `SingleCoreSend` is the right bound to use and it's OK to
use the `SingleCoreMutex`.

### Blockers

The `SingleCore*` API depends on the `auto trait` feature, which as of Rust
1.38.0 is still unstable (gated behind the `optin_builtin_traits` feature gate).
Thus the implementation of this API will need to wait until the feature is
stabilized. One could implement this API behind a Cargo feature that requires
nightly Rust but that would hinder its adoption so this proposal advises against
doing that.

## Intermediate step

Because the `SingleCore*` API is, time-wise, far away this document proposes, as
an intermediate step, landing an `unsafe` `RacyMutex` type and deprecating /
removing `Mutex` *today*.

`RacyMutex` will pretty much replicate the API of `Mutex`, including its `Sync`
implementation, but will have an `unsafe` constructor. The safety contract of
the constructor is that the value has to be created in a single-core context;
only within that context the rest of the `RacyMutex` API will be sound.

The main parts of the `RacyMutex` API are shown below:

``` rust
// crate: bare-metal

/// A mutex that's racy in multi-core contexts
pub struct RacyMutex<T> { /* .. */ }

/// IMPORTANT: `RacyMutex` is only `Sync` in a single-core context
unsafe impl<T> Sync for RacyMutex<T> {}

impl<T> RacyMutex<T> {
    /// Creates a new `RacyMutex`
    ///
    /// # Safety
    ///
    /// By constructing a `RacyMutex` the caller is asserting that the value
    /// will only be used in a single-core context; this is the case, for
    /// example, if the constructor is called in a single-core application.
    ///
    /// Using `RacyMutex` in a multi-core context is unsound. For that reason,
    /// `RacyMutex` must NEVER be used in a general purpose library; it should
    /// only be used in (a) HALs for single-core devices, and (b) applications
    /// for single-core devices.
    pub const unsafe fn new(data: T) -> Self {
        // ..
    }


    // .. and the rest of the `Mutex` API ..
}
```

### Migration path

Existing applications that use `bare_metal::Mutex`-es in static variables can
easily migrate to `RacyMutex` by wrapping the constructors in an `unsafe` block
and replacing `Mutex` with `RacyMutex`.

``` rust
// this
// static SHALL_WE_DANCE: Mutex<RefCell<Option<Partner>>> = Mutex::new(/* .. */);

// becomes
static SHALL_WE_DANCE: RacyMutex<RefCell<Option<Partner>>> = unsafe {
    // NOTE(unsafe): this is a single-core application
    RacyMutex::new(/* .. */)
};

#[entry]
fn main() {
    // ..
}

#[exception]
fn SysTick() {
    interrupt::free(|cs| {
        // this stays the same (safe)
        let yes = SHALL_WE_DANCE.borrow(cs).borrow_mut().take().unwrap();
        // ..
    });
}
```

# Unresolved questions

- Bikeshed all names

- Should `RacyMutex` be removed once the `SingleCore*` API has been implemented?
  Or should both co-exist side by side?

# Alternatives

- Come up with a interrupt-masking / spinlock hybrid implementation for `Mutex`
  that's both single-core and multi-core `Sync`.

- Remove `bare_metal::Mutex` without replacement.
