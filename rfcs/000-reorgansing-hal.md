- Feature Name: Reorganising the Embedded HAL
- Start Date: 17-Dec-2019
- RFC PR: 
- Rust Issue: 

# Summary
[summary]: #summary

Currently when the Embedded HAL wants to roll out new version of a trait, it does so by embedded the version number into
the module name - e.g. `embedded_hal::digital::v2`. This approach exposes to much internal complexity to the end user and
doesn't give us anywhere to experiment with new features. This RFC proposes creating sub-crates to replace the current
modules, and replacing the embedded-hal crate with a simple crate that only re-exports stable types from the relevant
sub-crates.

# Motivation
[motivation]: #motivation

By reorganising the Embedded HAL crate into sub-crates, we gain a place to put experimental features. The barrier to entry
is lower, and by having them published on crates.io, it is easier to create proof-of-principle implementations and users
of those new traits. By re-exporting the types from a top-level embedded-hal trait, we retain backwawrds compatibility with
the existing design and have a well-defined set of stable traits people should be able to rely on for production use-cases.

# Detailed design
[design]: #detailed-design

To take digital GPIO pins as an example:

1. Take the embedded_hal::digital::v2 module and turn it into an embedded-hal-digital crate.
2. Publish embedded-hal-digital to crates.io as v1.0.
3. Add a `pub use embedded_hal_digital::OutputPin` to embedded-hal::digital (and embedded_hal::digital::v2 to retain compatibility).
4. Take PRs on embedded-hal-digital as new features are developed (e.g. an BiDirectionalPin, or a TriStatePin).
5. Publish embedded-hal-digital as required, noting that it is an unstable crate that receives a large number of major version bumps and you should be prepared for breakage.
6. Up-version the embedded-hal-digital dependency in embedded-hal as and when we determine that some new trait is now stable, and `pub use` the trait in the top level crate.

Repeat for serial, i2c, spi, etc, plus all their blocking variants (which will live in the same sub-crate as the non-blocking crate for simplicity).

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

We would need to update:

1. The Rust Embedded Book.
2. The examples in the Discovery book.
3. The embedded-hal crate and its documentation.
4. Inform the developers of the nrfXXX-hals, tm4c12x-hals and the stm32-xxx hals (and any else we're aware of) of this change.

New users should see very little change - they can just use `embedded_hal::digital::OutputPin` as they do now (but without having to deal with the `v2` and `v3` details).

# Drawbacks
[drawbacks]: #drawbacks

This is a big change to embedded-hal. People may come to rely on the 'unstable' crates without realising they are unstable.

# Alternatives
[alternatives]: #alternatives

1. We continue to add `digital::v3`, `digital::v4` as we keep breaking the Digital IO pin API.
2. We add lots of new APIs to embedded-hal, but hidden behind an `unstable` feature that most people just turn on because "that's where the good stuff is".

# Unresolved questions
[unresolved]: #unresolved-questions

Not sure at this point. Something will come to me I'm sure.

