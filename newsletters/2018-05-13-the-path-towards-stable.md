# Cortex-M library development now possible on beta and the path towards stable embedded Rust

> 2018-05-13

> **TL;DR**
>
> * 1.27 = embedded (Cortex-M) *library* development on stable
> * 1.28 or 1.29 = embedded (Cortex-M) *application* development on stable
> * We are making breaking changes in the ecosystem in order to move towards development on the
>   stable channel
> * If you are a crate maintainer / author, make sure you test your crates with the new versions of
>   `cortex-m` (and similar) on the current `beta` release of Rust
> * If the latest version of a dependency doesn't compile on `beta`, file an issue and/or ping us on
>   the #rust-embedded IRC channel
> * Thanks for your patience!

We are happy to announce that *library* development for the Cortex-M targets is now possible on the
beta channel! :tada:

For example, to cross compile a library crate for the Cortex-M3 you would run these commands:

``` console
$ # switch to the beta channel
$ rustup default beta

$ # install the rust-std component (pre-compiled core) for the target
$ rustup target add thumbv7m-none-eabi

$ # get some source to build
$ cargo clone cortex-m --vers 0.5.0 && cd cortex-m

$ # then It Just Works
$ cargo build --target thumbv7m-none-eabi
```

What about embedded *application* development? There is just a single unstable feature that prevents
us from building a `no_std` binary on stable: the `panic_fmt` lang item. This unstable feature will
soon be removed in favor of the `#[panic_implementation]` feature ([implementation PR]), which is
slated for stabilization in time for the edition release (we are hoping to stabilize it as early as
1.28 but we'll see).

[implementation PR]: https://github.com/rust-lang/rust/pull/50338

## Making the Cortex-M ecosystem work on stable

"`no_std` binaries are *possible* on stable" is very different from "*my* embedded application
builds on stable". The later requires all the dependencies to also compile on stable. So we are
starting to migrate the Cortex-M ecosystem to work on stable. All the crates listed below are now
compiling on the beta channel.

Legend: `crate-name` - link to CHANGELOG - link to stabilization PR - ~removed unstable features~

- [`cortex-m-quickstart`][c0] - [CHANGELOG][ch0] - [PR][p0]

[c0]: https://docs.rs/cortex-m-quickstart/~0.3
[ch0]: https://github.com/japaric/cortex-m-quickstart/blob/master/CHANGELOG.md#v030---2018-05-12
[p0]: https://github.com/japaric/cortex-m-quickstart/pull/29

- [`cortex-m-rt`][c1] - [CHANGELOG][ch1] - [PR][p1] - ~asm!~ ~core_intrinsics~ ~global_asm!~
  ~lang_items~ ~linkage~ ~naked_functions~ ~used~

[c1]: https://crates.io/crates/cortex-m-rt/0.5.0
[ch1]: https://github.com/japaric/cortex-m-rt/blob/master/CHANGELOG.md#v050---2018-05-12
[p1]: https://github.com/japaric/cortex-m-rt/pull/69

- [`cortex-m-semihosting`][c2] - [CHANGELOG][ch2] - [PR][p2]

[c2]: https://crates.io/crates/cortex-m-semihosting/0.3.0
[ch2]: https://github.com/japaric/cortex-m-semihosting/blob/master/CHANGELOG.md#v030---2018-05-10
[p2]: https://github.com/japaric/cortex-m-semihosting/pull/16

- [`cortex-m`][c3] - [CHANGELOG][ch3] - [PR][p3] - ~asm!~ ~const_fn~

[c3]: https://crates.io/crates/cortex-m/0.5.0
[ch3]: https://github.com/japaric/cortex-m/blob/master/CHANGELOG.md#v050---2018-05-11
[p3]: https://github.com/japaric/cortex-m/pull/88

- [`embedded-hal`][c4] - [CHANGELOG][ch4] - [PR][p4] - ~never_type~

[c4]: https://crates.io/crates/embedded-hal/0.2.0
[ch4]: https://github.com/japaric/embedded-hal/blob/master/CHANGELOG.md#v020---2018-05-12
[p4]: https://github.com/japaric/embedded-hal/pull/80

- [`f3`][c5] - [CHANGELOG][ch5] - [PR][p5]

[c5]: https://crates.io/crates/f3/0.6.0
[ch5]: https://github.com/japaric/f3/blob/master/CHANGELOG.md#v060---2018-05-12
[p5]: https://github.com/japaric/f3/pull/93

- [`l3gd20`][c6] - [CHANGELOG][ch6] - [PR][p6] - ~Unsized~

[c6]: https://crates.io/crates/l3gd20/0.2.0
[ch6]: https://github.com/japaric/l3gd20/blob/master/CHANGELOG.md#v020---2018-05-12
[p6]: https://github.com/japaric/l3gd20/pull/4

- [`lsm303dlhc`][c7] - [CHANGELOG][ch7] - [PR][p7] - ~Unsized~

[c7]: https://crates.io/crates/lsm303dlhc/0.2.0
[ch7]: https://github.com/japaric/lsm303dlhc/blob/master/CHANGELOG.md#v020---2018-05-12
[p7]: https://github.com/japaric/lsm303dlhc/pull/4

- [`stm32f103xx`][c8] - [CHANGELOG][ch8] - [PR][p8] - ~const_fn~ ~global_asm!~ ~try_from~
  ~use_extern_macros~ ~used~

[c8]: https://crates.io/crates/stm32f103xx/0.10.0
[ch8]: https://github.com/japaric/stm32f103xx/blob/master/CHANGELOG.md#v0100---2018-05-12
[p8]: https://github.com/japaric/stm32f103xx/pull/24

- [`stm32f30x-hal`][c9] - [CHANGELOG][ch9] - [PR][p9] - ~never_type~

[c9]: https://crates.io/crates/stm32f30x-hal/0.2.0
[ch9]: https://github.com/japaric/stm32f30x-hal/blob/master/CHANGELOG.md#v020---2018-05-12
[p9]: https://github.com/japaric/stm32f30x-hal/pull/25

- [`stm32f30x`][c10] - [CHANGELOG][ch10] - [PR][p10] - ~const_fn~ ~global_asm!~ ~try_from~
  ~use_extern_macros~ ~used~

[c10]: https://crates.io/crates/stm32f30x/0.7.0
[ch10]: https://github.com/japaric/stm32f30x/blob/master/CHANGELOG.md#v070---2018-05-12
[p10]: https://github.com/japaric/stm32f30x/pull/16

- [`svd2rust`][c11] (output of) - [CHANGELOG][ch11] - [PR][p11] - ~const_fn~ ~global_asm!~
  ~try_from~ ~use_extern_macros~ ~used~

[c11]: https://crates.io/crates/svd2rust/0.13.0
[ch11]: https://github.com/japaric/svd2rust/blob/master/CHANGELOG.md#v0130---2018-05-12
[p11]: https://github.com/japaric/svd2rust/pull/203

There's a lot of breaking changes there but the most visible one is `cortex-m-rt` because it changes
how applications are structured. Check the [`cortex-m-quickstart`][c0] template for instructions on
how to set up an embedded application using the latest versions of everything.

There are still a lot of crates that will need to be updated to work on beta. Among the more general
purpose ones we have [`heapless`] and [`cortex-m-rtfm`], which needs to be update to work with
`cortex-m-rt` v0.5.0.

[`heapless`]: https://crates.io/crates/heapless
[`cortex-m-rtfm`]: https://crates.io/crates/cortex-m-rtfm

### How you can help us

Try to compile your Cortex-M crate using the beta channel! If your crate is an application the build
will certainly fail, but check if all the dependencies, excluding the [panic handler crate][phc],
compile on beta. If any dependency doesn't compile on the beta channel let the author know, but
first check if there's a new version of the crate that already compiles on the beta channel. And if
you feel up to the task you could even send a PR to the dependency to make it compile on beta -- the
list of stabilization PRs in the previous section may give you a clue on how to move the crate to
beta.

If you are the author of a Cortex-M device crate, a crate generated using `svd2rust`, moving it to
stable only requires regenerating it using `svd2rust` v0.13.0. Do note that the generation process
[has changed] for the Cortex-M target; also make sure you bump the minor version when you publish a
new version.

[has changed]: https://docs.rs/svd2rust/0.13.0/svd2rust/#usage

If you are the author of a driver crate or a HAL implementation, please update your crate to use
v0.2.0 of `embedded-hal` and test that it builds on beta. Make sure you bump the minor version when
you publish a new version of your crate -- bumping the version of the `embedded-hal` dependency is a
breaking change (drivers that depend on `embedded-hal` v0.1.0 can't be used with HAL implementation
crates that depend on `embedded-hal` v0.2.0).

If you are porting an application to the newest `cortex-m-*` crates you may hit these errors:

- "error: requires `start` lang_item" on binary crates. You need to switch from the standard `main`
  interface to `#![no_main]` + `entry!`. Use the [examples][qse] in the quickstart template as a
  reference.

[qse]: https://docs.rs/cortex-m-quickstart/0.3.0/cortex_m_quickstart/examples/_0_minimal/index.html

- "error[E0015]: calls in statics are limited to constant functions, tuple structs and tuple
  variants". `const fn` is not a stable feature so it's not used in the API by default. The
  `cortex-m` provides a `"const-fn"` feature to opt into this unstable feature and expose `const`
  functions in the API; other dependencies should / probably provide a similar feature.

## What does this all mean for you?

To those of you who have been doing embedded development on nightly: things will get *much easier*
from now on. We have reduced the number of unstable features in core crates of the Cortex-M
ecosystem to **zero**. Although you will have to continue to use nightly for application development
you can expect zero breakage from these crates when updating the nightly compiler. There is one
expected breakage coming soon though: `panic_fmt` will be removed as announced above. This will
break the [panic handler crates][phc]; however, the breakage will be isolated to those crates and
the fix will consist of simply updating the dependency version.

[phc]: https://crates.io/keywords/panic-impl

To those of you who have been meaning to dive into embedded Rust development: we'll soon be done
with our "embedded Rust on stable" tasks and we'll move our focus to documenting the embedded
development process, tooling and ecosystem. The [embedded Rust book][book] will become the resource
to get started on embedded Rust, but it doesn't quite exist right now.

[book]: https://github.com/rust-lang-nursery/embedded-wg/blob/master/books/embedded-rust-book/README.md

## What about architectures other than ARM Cortex-M?

The path carved for ARM Cortex-M can be followed by the other architectures (ARM Cortex-R, AVR,
MSP430, RISCV, etc.). Each architecture presents its own extra set of challenges though:

- MSP430 being the second oldest embedded / `no_std` target is the closest one to make it to stable
  (as a tier 2 target) by the edition release. The main blocker is that `extern "msp430-interrupt"`
  is not available on stable but I think that may be possible to work around using external C /
  assembly files + FFI.

- Both AVR and RISCV are blocked from being included in the Rust compiler due to either the
  immaturity of the LLVM backend or LLVM backend bugs. Not many (embedded) Rust developers can also
  hack LLVM backends so we are short handed on the AVR front; OTOH, the RISCV is being actively
  developed by third parties so it's probably just a matter of time before we add it to rustc.

- ARM Cortex-R LLVM backend is probably in good shape due to the similarities between its
  instruction set and the ARM Cortex-M instruction set, so adding the target to the compiler should
  be straightforward. However, the crate ecosystem is nonexistent at this point so there's a lot of
  work to be done on that front.

---

This work is part of the [embedded WG][ewg] effort towards [making embedded development possible on
the stable channel][stable]. (We are almost done! :tada:)

[ewg]: https://github.com/rust-lang-nursery/embedded-wg
[stable]: https://github.com/rust-lang-nursery/embedded-wg
