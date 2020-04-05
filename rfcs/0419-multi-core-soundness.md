- Feature Name: `multi_core_soundness`
- Start Date: 2020-01-26
- RFC PR: [#419](https://github.com/rust-embedded/wg/pull/419)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Change how we model multi-core MCUs in the following ways:

* Stop using `Send` and `Sync` to model anything related to cross-core
  interactions since those traits are unfit for the purpose.
* Declare that `Send` is not sufficient to transfer resources between cores,
  since memory addresses can have different meanings on different cores.
* Mutexes based on critical sections become declared sound due to the above:
  They can only turn `Send` data `Sync`, but neither allows cross-core
  interactions anymore.
* Punt on how to deal with cross-core communication and data sharing for now,
  and leave it to the ecosystem.

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
resources. It is likely that when the language semantics around these traits are
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
interrupt handlers, but runs on a single-core MCU: In both cases, all global
resources exist precisely once, and may be shared and exchanged between
interrupt handlers or threads (provided they implement `Send`/`Sync`) with no
risk of the resources changing their meaning when sent.

# Detailed design
[design]: #detailed-design

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

## `bare_metal::Mutex` is now sound

Since a `Mutex` only turns `Send` data into `Sync` data, it does not allow any
other cores in the system to access the protected data. Since disabling
interrupts suspends all but the current *execution context*, exclusive access
to the protected resource is now granted.

Providing a way to share data between cores fundamentally depends on the memory
layout of the device and application and is left to device-specific HALs,
support crates or frameworks like [µAMP].

[`bare-metal`]: https://github.com/rust-embedded/bare-metal

## The fate of Symmetric Multi-Processing (SMP) apps

In SMP apps, only a single executable is used for all cores, while each core can
have a separate entry point (or another mechanism of identifying the running
core). This means that all `static`s are shared by default.

Since defining a `static` only requires that its type is `Sync`, this would be
unsound. For example, it would allow storing a `bare_metal::Mutex` in a `static`
and access it from all cores. Therefore, this RFC foregoes the ability to write
safe SMP apps in Rust, instead proposing to shift focus to AMP apps, which do
not share data by default and produce a separate executable per core.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

(see above)

# Drawbacks
[drawbacks]: #drawbacks

* This makes writing applications for multi-core MCUs more difficult if there
  are no mature libraries for the target platform (that would provide APIs and
  traits for cross-core operations).

  While multi-core MCUs have become somewhat more common, it is expected that
  the vast majority of embedded Rust users will continue to use single-core
  MCUs. For those users, providing a sound and safe-to-use `Mutex` that works by
  disabling interrupts is beneficial. Even for multi-core applications, it is
  expected that the actual cross-core communication is limited to a small number
  of places in the code, so making it more difficult has limited impact.

* This RFC generally rules out SMP apps that run the same firmware image on
  multiple cores. These would be able to share data via `static`s, which only
  requires a `Sync` bound, and that is not sufficient to guarantee safe
  operation when accessed from multiple cores.

# Alternatives
[alternatives]: #alternatives

* Accept [RFC 388] instead, introducing a `SingleCore{Send,Sync}` auto trait
  pair once auto traits are stable, and make `Mutex::new` an `unsafe fn` in the
  interim. Remove unsound `Send` impls from peripherals.

* Do what this RFC proposes, and also introduce `CoreSend`/`CoreSync` traits to
  model cross-core interaction. (This was what this RFC initially proposed, but
  it was decided that we should focus on fixing the soundness issues first and
  leave multi-core support to be implemented outside the core ecosystem.)

# Unresolved questions
[unresolved]: #unresolved-questions

* None so far.

[`bare_metal::Mutex`]: https://docs.rs/bare-metal/0.2.5/bare_metal/struct.Mutex.html
[RFC 388]: https://github.com/rust-embedded/wg/pull/388
