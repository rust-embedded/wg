# RFC: Add a Mutex trait

# Summary
[summary]: #summary

This RFC proposes to add a `mutex-trait` crate which adds a `Mutex` trait for a stable and generic foundation on which HALs and applications can be built on when there is a need for mutual exclusive access.

The original idea to a `Mutex` trait was given here [embedded-hal#132].

[embedded-hal#132]: https://github.com/rust-embedded/embedded-hal/pull/132

# Motivation
[motivation]: #motivation

Today in the embedded ecosystem there is no base on which mutual exclusive access stems from which can be used as a generic. This has made it so there are now multiple implementations of mutexes in the ecosystem that are incompatible (cortex-m and risc-v mutex based on [bare-metal mutex], [RTFM mutex], [mutex.rs], ...). The consequence of this is that no HAL or application can be made independent on the choice of `Mutex` and lock within their implementations, one has to select one which is detrimental to generic implementations.

This can however easily be addressed by introducing a `Mutex` trait on which previous implementations can be built upon, which each architectural crate (cortex-m, risc-v, etc) reexports/implements, together with the [`embedded-hal`] which can use said trait in HAL design.

[bare-metal mutex]: https://github.com/japaric/bare-metal/blob/a2f8b222a14253141bbc7853e5699641e61e5ebf/src/lib.rs#L63
[RTFM mutex]: https://github.com/japaric/cortex-m-rtfm/blob/fafeeb27270ef24fc3852711c6032f65aa7dbcc0/src/lib.rs#L289
[mutex.rs]: https://gist.github.com/mvirkkunen/cc63d20648737022a8c5f93666093f4a
[`embedded-hal`]: https://github.com/rust-embedded/embedded-hal

# Detailed design
[detailed-design]: #detailed-design

The trait is proposed as follows:

```rust
pub trait Mutex {
    /// Data protected by the mutex
    type Data;

    /// Creates a critical section and grants temporary access to the protected data
    fn lock<R>(&mut self, f: impl FnOnce(&mut Self::Data) -> R) -> R;
}
```

The design is such that the data protected by the mutex is only accessible within the closure provided to the `lock` method, moreover the closure shall run within a critical section such that it is guaranteed that no-one else can access the same data at the same time. The critical section can for example be:

* Disabling interrupts (cortex-m / risc-v critical sections)
* Analyzing the program and set BASEPRI in Cortex-M devices (Stack Resource Policy/RTFM way)
* etc

### Design decisions and compatibility

####  The [Rust standard library `Mutex`] uses `&self` for the `lock` method, why choose `&mut self`?

This is the **most important** part of this RFC proposal. Let's assume that the implementation was `&self`, what would this mean? This would allow the following code to compile, which is unsound by construction:

```rust
fn unsound(mtx: &impl mutex_trait::Mutex<Data = i32>) {
    mtx.lock(|data| {
        *data += 1;

        mtx.lock(|data| *data += 1);
    });
}
```

This example allows for a mutex to lock itself within a lock of itself, which is **unsound by construction** and creates an immediate deadlock - while if implementing it with `&mut self` the following would happen:

```rust
fn sound(mtx: &mut impl mutex_trait::Mutex<Data = i32>) {
    mtx.lock(|data| {
        *data += 1;

        mtx.lock(|data| *data += 1);
    });
}
```

Which produces the following error and making the mutex **sound by construction**:

```
error[E0499]: cannot borrow `mtx` as mutable more than once at a time
  --> src/main.rs:40:5
   |
40 |       mtx.lock(|data| {
   |       ^   ---- ------ first mutable borrow occurs here
   |       |   |
   |  _____|   first borrow later used by call
   | |
41 | |         *data += 1;
42 | |
43 | |         mtx.lock(|data| *data += 1);
   | |         --- first borrow occurs due to use of `mtx` in closure
44 | |     });
   | |______^ second mutable borrow occurs here
```

The common point raised with `&mut self` is that this definition is **not compatible** with with the current `Mutex<RefCell<T>>` way of implementing mutexes in embedded Rust today, as one needs to take a mutable reference to the mutex. This can however be circumvented with the following implementation of the trait:

```rust
struct Mutex<T> {
    data: T,
}

impl<T> Mutex<T> {
    pub const fn new(data: T) -> Self {
        Self { data }
    }

    fn access(&self, _cs: &CriticalSection) -> &T {
        &self.data
    }
}

impl<'a, T> mutex_trait::Mutex for &'a Mutex<RefCell<T>> {
    type Data = T;

    fn lock<R>(&mut self, f: impl FnOnce(&mut Self::Data) -> R) -> R {
        cortex_m::interrupt::free(|cs| f(&mut *self.access(cs).borrow_mut()))
    }
}
```

Which allows for the following program to exist, hence **not breaking compatibility** with existing pattern:

```rust
static MUT: Mutex<RefCell<i32>> = Mutex::new(RefCell::new(0));

#[entry]
fn init() -> ! {
    let mut r = &MUT; // Note that r is mutable

    // Locking!!
    r.lock(|data| *data += 1);

    loop {}
}

#[interrupt]
fn SWI0_EGU0() {
    let mut r = &MUT; // Note that r is mutable

    // Locking!!
    r.lock(|data| *data += 1);

    // More Locking!!
    increment_in_mutex(&mut r);
}

// Look no RefCell and a lock!!!
fn increment_in_mutex(m: &mut impl MutexTrait<Data = i32>) {
    m.lock(|data| *data += 1);
}
```

This implementation does however **allow for relocking the mutex within a lock again, so why bother** (double lock = panic in this case)? The use of `&mut self` locks has been evaluated extensively in the [RTFM framework], and is the entire basis for safe by construction applications. When pairing the `Mutex` trait with procedual macros and analysis it is possible to create deadlock and race condition free programs by construction - **but these are only possible** if we have a basis which is build on the `&mut self`.

It is, in my opinion, an error that the standard library did not use this design together with the class of issues it eliminates.

For a detailed analysis on `&mut self` critical sections see [this part of the RTFM book].

[Rust standard library `Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html#method.lock
[RTFM framework]: https://github.com/japaric/cortex-m-rtfm
[this part of the RTFM book]: https://github.com/japaric/cortex-m-rtfm/blob/2596ea0e46bec73d090d9e51d41e6c2f481b8e15/book/en/src/internals/critical-sections.md

#### Why is the result of the `lock` method not a `Result<...>`?

The `Mutex` trait is meant to be available so that HALs (and other crates/apps) can have a notion about a resource and unique access to it. For example, this means we can make driver which has shared access we can protect this driver via a `lock` and we can reason about safety and soundness of said driver.

And here comes the problem with a `Result` based `lock` - **what does it mean to have a lock which can fail**? A lock which can fail would be difficult to use, if not impossible, to create HALs as the reason for the lock failing is outside the scope of the HAL - only an infallible lock makes sense here. Another view is what if a lock fails - what does error handling mean at that low level rather than just allowing it to boil up through the API? Or, if you have half setup a peripheral and a `lock` fails, how do you abort? How do you reason about Worst Case Execution Times? What state is the hardware left in? Suddenly drivers have to implement mitigation and error handling routines that are **system design flaws**.

The next thing is, **why should a lock be fallible**? One implementation of a spin-lock mutex was found which used a `Result` return to indicate a deadlock, but what other than a panic is correct if one would get a deadlock in a driver?

All these questions point to an infallible `Mutex` trait. The example of a deadlock in a driver is to simply implement this trait and `.unwrap()` in the implementation.

It boils down to this philosophy: `panic` is for program error - `Result` is for user error

#### How does one implement generic functions that understand `lock`?

The following example shows how to write generic functions over the trait:

```rust
// A simple resource holding an `i32`
fn increment_in_mutex(mtx: &mut impl mutex_trait::Mutex<Data = i32>) {
    mtx.lock(|data| *data += 1);
}
```

## Rules for implementations

* Any object implementing the `Mutex` trait guarantees exclusive access to the data contained within the mutex for the duration of the lock.

## FAQ

> Can this be used in a lock/unlock setting such as:
>
> ```rust
> mtx.lock();
>
> // Do something ...
>
> mtx.unlock();
> ```

No. The `Mutex` trait is specifically for scoped use, i.e. the lock is defined in such a way that the resource protected by the lock is only accessible within the critical section that is the closure in the `lock` method.

This means that for example if you have a peripheral driver and you want to lock access to it while it does its work, this mutex is not designed for that. However you can implement a higher level mutex on top of this one.

> Should the `Mutex` trait not be in the [`embedded-hal`]?

It is our strong recommendation to **not** place it in the [`embedded-hal`] crate.
If there are breaking changes in the [`embedded-hal`] it can be so that competing implementations of the Mutex trait starts to exist which could bifurcate the ecosystem based on version, plus that the Mutex trait will be tied to HAL releases.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

There will be 2 reference implementations after the acceptance of this RFC. One direct implementation based on critical sections in `cortex-m` and `risc-v` crates (that now reexport the [bare-metal mutex]) as the mutex will now be implemented in these crates and the reexport removed. And another in the [RTFM framework] which works together with procedural macros to give correct by construction resource handling and locks.

# Drawbacks
[drawbacks]: #drawbacks

As of writing this RFC I see no drawbacks.

# Alternatives
[alternatives]: #alternatives

The alternatives, taking `&self` and returning a `Result<...>` are discussed in [Detailed design](#detailed-design).

One can also futher follow the [Rust standard library `Mutex`] and its RAII pattern, however this should not be persued dues to these causes:

1. The RAII version can be used to implement the closure and hence the closure can be implemented with an `std`-like mutex in the background.
2. The closure is explicit where it starts and ends, no need to wait for the destruction or add extra `{ ... }` around code to get predictable unlocks.
3. Makes code using locks more stable under refactoring, as small changes can drastically change the scope of a lock when using RAII.
4. A RAII based mutex can be `mem::forget` or `mem::swap` and all guarantees are out, the closure based mutex cannot be circumvented easily.
5. The RAII based mutex can be circumvented easily as follows:

```rust
// Break all guarantees
let x = lock1.lock();
let y = lock2.lock();
drop(x);
drop(y);
```

# Unresolved questions
[unresolved]: #unresolved-questions

* Should the trait be unsafe to indicate that one needs to be aware of the design constraints to implement it?
* Should the crate be named `mutex-trait` or something different?
