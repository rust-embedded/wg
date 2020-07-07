# Metadata

* Task Shepherd: @adamgreig
* Contributors: Newcomers welcome!
* Relevant repository: All WG repositories
* Status: Completed! All appropriate WG repos now have an MSRV.
  See the [tracking issue](https://github.com/rust-embedded/wg/issues/445)
  for more details.

# Background

All of the Embedded WG's projects should have a defined "Minimum Support Rust
Version" (MSRV). This is the oldest version of the Rust compiler we guarantee
will work with the project. Our MSRV policy is defined
[in ops/msrv.md](https://github.com/rust-embedded/wg/blob/master/ops/msrv.md),
along with how we should document and test it.

However, many of our projects do not document or test an MSRV, which means we
effectively do not know what the MSRV is and might accidentally raise it at
any time. This has been flagged recently in a few projects where the MSRV was
surprisingly high and untested.

# Suggested Task

This project would ensure all WG repositories which:

* Contain a Rust project
* Test that Rust project using a CI system

define a specific MSRV, test it automatically, and document it. The MSRV should
be defined in line with the WG policy of being at least one less than the
current stable Rust version.

# Detailed Work

Our specific goal is to ensure that each project:

* Documents the MSRV in the crate-level documentation, for example in `lib.rs`:

```markdown
# Minimum Supported Rust Version (MSRV)

This crate is guaranteed to compile on stable Rust 1.31 and up. It *might*
compile with older versions but that may change in any new patch release.
```

* Tests the MSRV in CI, for example:

```yaml
matrix:
  include:
    # MSRV
    - env: TARGET=thumbv7-none-eabi
      rust: 1.31.0
    # ...
```

These repositories have been identified as meeting the project criteria:

* [ ] [alloc-cortex-m](https://github.com/rust-embedded/alloc-cortex-m)
* [ ] [arm-dcc](https://github.com/rust-embedded/arm-dcc)
* [ ] [bare-metal](https://github.com/rust-embedded/bare-metal)
* [ ] [cargo-binutils](https://github.com/rust-embedded/cargo-binutils)
* [ ] [cortex-a](https://github.com/rust-embedded/cortex-a)
* [ ] [cortex-m-rt](https://github.com/rust-embedded/cortex-m-rt)
* [ ] [cortex-m-semihosting](https://github.com/rust-embedded/cortex-m-semihosting)
* [ ] [cortex-m](https://github.com/rust-embedded/cortex-m)
* [ ] [cortex-r](https://github.com/rust-embedded/patterns)
* [ ] [cross](https://github.com/rust-embedded/cross)
* [ ] [embedded-hal](https://github.com/rust-embedded/embedded-hal)
* [ ] [fixedvec-rs](https://github.com/rust-embedded/fixedvec-rs)
* [ ] [gpio-cdev](https://github.com/rust-embedded/gpio-cdev)
* [ ] [gpio-utils](https://github.com/rust-embedded/gpio-utils)
* [ ] [itm](https://github.com/rust-embedded/itm)
* [ ] [linux-embedded-hal](https://github.com/rust-embedded/linux-embedded-hal)
* [ ] [msp430-rt](https://github.com/rust-embedded/msp430-rt)
* [ ] [msp430](https://github.com/rust-embedded/msp430)
* [ ] [mutex-trait](https://github.com/rust-embedded/mutex-trait)
* [ ] [panic-itm](https://github.com/rust-embedded/panic-itm)
* [ ] [panic-semihosting](https://github.com/rust-embedded/panic-semihosting)
* [ ] [r0](https://github.com/rust-embedded/r0)
* [ ] [register-rs](https://github.com/rust-embedded/register-rs)
* [ ] [riscv-rt](https://github.com/rust-embedded/riscv-rt)
* [ ] [riscv](https://github.com/rust-embedded/riscv)
* [ ] [rust-i2cdev](https://github.com/rust-embedded/rust-i2cdev)
* [ ] [rust-spidev](https://github.com/rust-embedded/rust-spidev)
* [ ] [rust-sysfs-gpio](https://github.com/rust-embedded/rust-sysfs-gpio)
* [ ] [rust-sysfs-pwm](https://github.com/rust-embedded/rust-sysfs-pwm)
* [ ] [svd2rust](https://github.com/rust-embedded/svd2rust)
* [ ] [svd](https://github.com/rust-embedded/svd)

# Discussion / Unresolved Questions

* Do we want to also include a MSRV badge in repository READMEs? See
  [rand](https://github.com/rust-random/rand) for an example.
* Do we want to update the WG policy to stable-2, i.e.
  [#427](https://github.com/rust-embedded/wg/issues/427)?

# References

* WG MSRV policy: [msrv.md](https://github.com/rust-embedded/wg/blob/master/ops/msrv.md)
* Original discussion about MSRV policy: [#285](https://github.com/rust-embedded/wg/issues/285)
* PR adding WG MSRV policy: [#304](https://github.com/rust-embedded/wg/pull/304)
* Issue around guidance for MSRV: [#382](https://github.com/rust-embedded/wg/issues/382)
* Recent issue around rethinking the WG MSRV: [#427](https://github.com/rust-embedded/wg/issues/427)
* PR adding MSRV to cortex-m-rt: [#262](https://github.com/rust-embedded/cortex-m-rt/pull/262)
* PR adding MSRV to cortex-m: [#159](https://github.com/rust-embedded/cortex-m/pull/159)
