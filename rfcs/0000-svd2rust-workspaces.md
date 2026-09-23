# Svd2Rust workspace generation

- Feature Name: svd2rust_ws_gen
- Start Date: 2019-02-07
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)


## Summary

`svd2rust` is a tool to generate Rust code from svd-files.
This allows easier development of embedded code for certain platforms and µ-controllers.
There is a lot of code that is generated, sometimes upwards of a million lines of Rust.
Currently `svd2rust` does this as a single crate, using modules to seperate different peripherals from each other.

This RFC proposes a change to `svd2rust` to generate a workspace of crates, instead of just a single one.

## Motivation

Splitting up the code that is generated allows for ergonomics improvements in several dimensions.


1. Users would be able to pick-and-choose crates they want to use, i.e. only supporting certain peripherals
2. Using crates instead of modules allows `rustc` and `cargo` to use multiple codegen units,
   making compiles parallel by default, using system resources more effectively.

Especially decreased compile-times should be considered the primary motivation of this change.


## Guide-level explanation

Changes to `svd2rust` would mostly be under the hood. 
As such, interaction with `svd2rust` after the change aren’t much different from how they are now. 

Users can still just pull in a platform crate as a dependency → get all peripherals

```toml
    [project]
    # ...
    
    [dependencies]
    acme_mcu8008 = "1.1.0" # Includes _all_ peripherals
```

Additionally it’s possible to depend on a specific sub-set of peripherals

  - Pull in common platform crate
  - Only get access to specific peripherals

```toml
    [project]
    # ...
    
    [dependencies]
    acme_mcu8008_ethernet = "1.1.0"
    acme_mcu8008_spi = "1.1.0"
    acme_mcu8008_wom = "1.1.0"
```

The usage of modules `ethernet`, `spi` and `wom` would then only differ by their top-level import name.

```rust
    use acme_mcu8008::{ethernet, spi, wom};
    
    // vs
    
    use acme_mcu8008_ethernet as ethernet;
    use acme_mcu8008_spi as spi;
    use acme_mcu8008_wom as wom;
```

## Reference-level explanation

From a high-level perspective there are a few structural changes.

Each platform has a super-crate, same as now (i.e. `acme_mcu8008`).
Internally it exports (`pub use <peripheral`) all sub-crates.

All current peripheral modules are turned into sub-crates,
prefixed with the plaform name (i.e. `acme_mcu8008_ethernet`).

Additionally a `<platform>_commons` crate is generated to avoid circular dependency issues.
This crate would never need to be included by an end-user of the library directly

## Drawbacks

A change like this comes with some drawbacks in different domains. 

- Increased complexity of `svd2rust` code generator itself
- How to teach the interaction between the platform super-crate and peripheral-specific crates
- When using `codegen-units=1` in release mode, this could have impact on compile-times
- Possibly lower optimisation potential because of using multiple codegen units

Especially the last two points would require some proper profiling rather than speculation
to come to a conclusion about how much performance impact (and in what direction) this feature will have.


## Rationale and alternatives

An alternative to generating multiple crates for peripherals was to generate feature flags for each peripheral,
meaning that developers would have to activate feature flags on a HAL crate in order to be able to use certain peripheral modules.
This was partially implemented by [PR #236](https://github.com/rust-embedded/svd2rust/pull/236).


## Prior art

Two partially working implementation for feature gate switching on peripheral modules exist as 
[PR #236](https://github.com/rust-embedded/svd2rust/pull/236) and [PR #254](https://github.com/rust-embedded/svd2rust/pull/254). 
An unpublished version of `svd2rust` workspace generation exists and highlighted a few issues that have
influenced the design of this RFC.
A tool to handle workspace-synchronised releases has a partially working implementation
[here](https://github.com/spacekookie/cargo-ws-release).


## Unresolved questions

A HAL crate could potentially have dozens of peripheral sub-crates.
This will result in a lot more traffic on crates.io, as well as causing a lot more document builds on docs.rs

We would have to coordinate with both of those teams in order to find a good strategy of how to handle this additional load.


