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

* Add `digital::v3` interface with the same traits as `digital::v2`, but with methods named with `try_` prefix.
For example, `ToggleableOutputPin` will have `try_toggle` method instead of `toggle`.
* Provide transparent conversion from `digital::v2` traits to `digital::v3` traits.
* Change `digital::v1` compatibility layer to convert from `digital::v3` traits instead of `digital::v2` ones.
* Add `V2OutputPin` proxy to the `digital::v2` compatibility layer to provide `digital::v3` -> `digital::v2` downgrade.
* Add `digital::v3` traits to prelude.
* Un-deprecate `digital::v1` traits, add a note for driver developers.
* Deprecate `digital::v2` traits in favor of `digital::v3` traits.
* Move HALs back to `digital::v1` interface.

### Consequences

* End users will be able to use both fallible and infallible traits and import them via `embedded-hal` prelude.
* HALs will export GPIOs as infallible (as they really are).
* Driver developers can move to fallible `digital::v3` traits with minimal effort.
* End users that use drivers with `digital::v2` interface will continue using them without any changes in
most of the cases. Newer `digital::v3` objects could be passed to `digital::v2` consumers via `V2OutputPin` shim.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this



What names and terminology work best for these concepts and why? 
How is this idea best presented—as a continuation of existing Rust patterns, or as a wholly new one?

Would the acceptance of this proposal change how Rust is taught to new users at any level? 
How should this feature be introduced and taught to existing Rust users?

What additions or changes to the Rust Reference, _The Rust Programming Language_, and/or _Rust by Example_ does it entail?

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
