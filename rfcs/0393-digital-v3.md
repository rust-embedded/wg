- Feature Name: digital_v3
- Start Date: 2019-11-17
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

This RFC proposes `digital::v3` fallible interface and un-deprecation of `digital::v1` interface.

# Motivation
[motivation]: #motivation

Current `digital::v2` interface has some problems:

* `digital::v2` interface has the same method names as `digital::v1` interface,
due to this fact they cannot be used together and `digital::v2` traits in prelude may break things.
* Current implementation deprecates `digital::v1` in favor of `digital::v2` interface,
this has unforeseen consequences for HAL libraries.
As a result, user code suffers from useless `.unwrap()` constructions.

This RFC aims to solve these problems.

Currently, `embedded-hal` digital traits are used from three perspectives:
* driver developers
* hal developers
* end-users

`digital::v2` traits were introduced to provide driver developers with a way to work with
potentially fallible GPIO pins. Such pins can be created by drivers for external GPIO expanders connected
via fallible peripherals (e.g. i2c). At the same time, most MCUs do not have fallible GPIOs and hals are not
required to implement fallible `digital::v2` interface for them. 

Due to the deprecation of `digital::v1` traits, hal maintainers were annoyed by deprecation warnings and
switched to `digital::v2` traits to get rid of these warnings. This change introduced another problem:
end-users started getting warnings about unused result for the new (now `digital::v2`) GPIO methods.
These warnings introduced the `.unwrap()` pattern for using GPIO methods, even for GPIOs provided by MCU hal
libraries. Since most embedded projects do not use external fallible GPIOs, this led to
a decrease in code readability.

From the end-user perspective, co-existence of `digital::v2` and `digital::v1` traits led to another problem:
since all `digital::v1` implicitly implement `digital::v2` traits, they implement two different
traits with the same method names. When both traits are in scope, calls to any of the `digital::v1` methods
produce an error due to the ambiguity of used trait. Due to this fact, one also can't have both traits in
`embedded-hal` prelude for convenience.

# Detailed design
[design]: #detailed-design

This RFC proposes the following changes:

* Add `digital::v3` with both fallible and infallible traits.
Fallible traits will provide the same interface as `digital::v2`, but will reside under `fallible` module and have
`try_` prefix for the trait methods.
For example, `ToggleableOutputPin` will have `try_toggle` method instead of `toggle`.
Infallible traits will be exactly the same as in `digital::v1` interface.

* Provide transparent conversion from `digital::v1` and `digital::v2` traits to `digital::v3` traits.
* Change `digital::v1` compatibility layer to convert from `digital::v3` traits instead of `digital::v2` ones.
* Add `V2OutputPin` proxy to the `digital::v2` compatibility layer to provide `digital::v3` -> `digital::v2` downgrade.
* Deprecate `digital::v2` traits in favor of `digital::v3` fallible traits.
* Switch from `digital::v1` to `digital::v3` default.
* Move HALs to infallible `digital::v3` interface where applicable.
* Survey published driver crates and make a concerted effort to pull them all up to `digital::v3`.
* Drop `digital::v2` and `digital::v3` completely (with the semver-trick to previous versions if possible).

### Consequences

* Code that implements `digital::v1` traits will be transparently switched to `digital::v3` infallible traits
or (in case of explicit `digital::v1` usage) this compatibility will be provided via blanket implementations.
* Code that uses `digital::v1` traits will not require any changes provided that `digital::v3` traits are in scope.
* End users will be able to use both fallible and infallible traits and import them via `embedded-hal` prelude.
* HALs will export GPIOs as infallible (as they really are).
* Driver developers can move to fallible `digital::v3` traits with minimal effort.
* For the first time both `digital::v1` and `digital::v2` interfaces can be used without any changes.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Intended use of each pin interface in different contexts should be clearly indicated in the `embedded-hal` docs.

# Drawbacks
[drawbacks]: #drawbacks

The proposed approach does not provide hal and driver developers with a warning about the interface they should use. 

# Alternatives
[alternatives]: #alternatives

Changing `digital::v2` interface to provide different method names in all the pin traits.
This solves the name clashing problem, but does not solve `digital::v1` deprecation problem.

# Unresolved questions
[unresolved]: #unresolved-questions

Naming the methods: while `try_set_high` and `try_toggle` seem ok, `try_is_set_high` name is a bit weird.

Handling multiple error types in drivers. This RFC does not attempt to solve this issue,
leaving everything as it is.
