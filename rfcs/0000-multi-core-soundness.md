- Feature Name: `multi_core_soundness`
- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Change how we model multi-core MCUs in the following ways:

* Stop using `Send` and `Sync` to model anything related to cross-core
  interactions since those traits are unfit for the purpose.
* Declare that `Send` is not sufficient to transfer resources between cores,
  since memory addresses can have different meanings on different cores.
* Introduce `CoreSend` and `CoreSync` traits to the [`bare-metal`] crate.
* Require multi-core PACs/HALs, frameworks and/or libraries to manually mark
  data that is safe to send across core boundaries.
* Mutexes based on critical sections become declared sound due to the above:
  They can only turn `Send` data `Sync`, but neither allows cross-core
  interactions anymore.

# Motivation
[motivation]: #motivation

This RFC shares its motivation with [RFC 388]: The current way we model
multi-core MCUs (which is just like multi-threaded Rust programs are modeled)
means that [`bare_metal::Mutex`] and its reexports in `cortex_m` and other
crates are unsound.

With the push towards a 1.0 embedded ecosystem, we need to make sure that we do
not expose unsound APIs like that, but also that our usage and understanding of
language semantics is in line with upstream Rust and cannot lead to soundness
issues down the road.

Additionally, it would be great to have a good story for multi-core MCUs, since
they are often difficult to develop for, while more and more vendors have
multi-core products available not just for specialized applications, but as
relatively general MCUs.

## Today's Soundness Issues
[todays-issues]: #todays-soundness-issues

The auto trait behavior of `Send` and `Sync` causes soundness issues even today:
Imagine a user-defined struct `S` that only contains some primitive types.
Clearly, this struct will and should automatically implement `Send` and `Sync`.
Now, because `S` implements `Sync`, `&S` will implement `Send`. This is simply
the languages definition of these traits and nothing we can change. Now a big
problem can arise when tranferring `&S` across core boundaries: Cores can have
private memory regions that are not accessible by other cores. If `T: Send` is
the only requirement for sending an object to another core, then this would not
be sound. Existing multi-core support via [µAMP] attempts to work around this
issue by requiring that `T: 'static`, but this approach turns out to be
[unsound][soundness-1].

In addition to that, peripherals (either in the processor core, or ones provided
by the MCU manufacturer) are generally `'static` and `Send` since being able to
transfer them to an interrupt handler is an extremely common, safe operation.
However, not all peripherals are available to all cores. In particular, *none*
of the Cortex-M core peripherals are.

[This blog post about µAMP][microamp-blog] outlines other issues and how they
were solved or worked around in that case. Note that µAMP has other known
[soundness issues][soundness-2] in addition to the one linked above.

[µAMP]: https://github.com/rtfm-rs/microamp
[microamp-blog]: https://blog.japaric.io/microamp/
[soundness-1]: https://github.com/rtfm-rs/microamp/issues/6
[soundness-2]: https://github.com/rtfm-rs/microamp#known-issues

## The Core Issue

To understand why `Send` is insufficient to transfer resource ownership across
cores, observe that multi-core MCUs behave quite differently from multi-threaded
programs (which are what `Send` is supposed to model): Each core can have
private peripherals that may be `Send`able between interrupt handlers, but that
do not exist (or, worse, something *else* exists at their address) on other
cores of the system. The same goes for core-local memory regions: All normal
operations are fine while they only happen on the core owning the memory
(including `Send`ing some memory to an interrupt handler).

Multi-threading in Rust was always modeled with the assumption that all threads
share the same resources, with `Send` and `Sync` only guarding *access* to those
resources. It is likely that when the language semantics around these traits is
more rigidly specified, this will cause a clearer mismatch with what multi-core
MCUs actually need.

A similar problem to that of multi-core MCUs exists when writing hosted Rust
applications: Inter-Process Communication (IPC). This is similar because while
processes may share memory, `Send` and `Sync` are unsuitable to model any
cross-process interaction since addresses and resources such as file descriptors
on different processes can have completely different meanings. At the same time,
it is important that `Send` and `Sync` *stay implemented* for both `File` and
heap-allocated types like `Box`, since sending them across thread boundaries is
still desired.

Something much closer to a multi-threaded program is one that makes use of
interrupt handlers: In both cases, all global resources exist precisely once,
and may be shared and exchanged between interrupt handlers or threads (provided
they implement `Send`/`Sync`) with no risk of the resources changing their
meaning when sent.

# Detailed design
[design]: #detailed-design

## Introduce common traits for modeling cross-core operations

Add the following trait definitions to [`bare-metal`]:

```rust
/// Types that can be transferred across core boundaries.
pub unsafe trait CoreSend {}

/// Types that can be accessed from multiple cores at once.
pub unsafe trait CoreSync {}
```

Note that these are not auto traits. That is not just because auto traits are
still an unstable feature, but also because that wouldn't be correct: As
outlined in [Today's Soundness Issues][todays-issues], whether some data is safe
to be sent between cores depends not just on its type contents, but also on its
location in memory.

This cannot be modeled with auto traits, but multi-core HALs and frameworks such
as [µAMP] can provide helpers that make dealing with this easier.

An important difference to Rust's `Send` and `Sync` traits is that there is *no*
blanket impl for `&T` when `T: CoreSync`. This is important since whether `&T`
is sendable between cores depends on whether `T` is stored in a memory region
that is accessible by *all* cores.

## `bare_metal::Mutex` is now sound

Since a `Mutex` only turns `Send` data into `Sync` data, it does not allow any
other cores in the system to access the protected data. Since disabling
interrupts suspends all but the current *execution contexts*, exclusive access
to the protected resource is now granted.

Providing a way to share data between cores fundamentally depends on the memory
layout of the device and application and is left to device-specific HALs,
support crates or frameworks like [µAMP].

[`bare-metal`]: https://github.com/rust-embedded/bare-metal

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

API documentation of `CoreSend` and `CoreSync` needs to be clarified. The
single-sentence documentation above is not meant to be complete.

## Document what `Send` and `Sync` mean

`Send` and `Sync` will be used only to model transfer and sharing of resources
between different *execution contexts* that run with the same fixed set of
global resources.

Here, an *execution context* is something that may execute code asynchronously
from other code (so without needing to be called). For example, a *thread* or
an *interrupt handler* would qualify as an *execution context*, while a
single-threaded futures executor would *not* create any more *execution
contexts* while it executes.

Concretely (for embedded Rust), that means `Send` and `Sync` can be used to
model threads in an RTOS, or interrupt handlers for bare-metal applications.


# Drawbacks
[drawbacks]: #drawbacks

This makes writing applications for multi-core MCUs more difficult if there are
no mature libraries for the target platform (that would provide APIs and traits
for cross-core operations).

While multi-core MCUs have become somewhat more common, it is expected that the
vast majority of embedded Rust users will continue to use single-core MCUs. For
those users, providing a sound and safe-to-use `Mutex` that works by disabling
interrupts is beneficial. Even for multi-core applications, it is expected that
the actual cross-core communication is limited to a small number of places in
the code, so making it more difficult.

# Alternatives
[alternatives]: #alternatives

Accept [RFC 388] instead, introducing a `SingleCore{Send,Sync}` auto trait pair
once auto traits are stable, and make `Mutex::new` an `unsafe fn` in the
interim. Remove unsound `Send` impls from peripherals.

# Unresolved questions
[unresolved]: #unresolved-questions

None so far.

[`bare_metal::Mutex`]: https://docs.rs/bare-metal/0.2.5/bare_metal/struct.Mutex.html
[RFC 388]: https://github.com/rust-embedded/wg/pull/388
