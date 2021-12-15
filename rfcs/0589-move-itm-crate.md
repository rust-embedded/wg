- Feature Name: move_itm_crate
- Start Date: 2021-12-01
- RFC PR: [#589](https://github.com/rust-embedded/wg/pull/589)

# Summary
[summary]: #summary

Archive [`rust-embedded/itm`](https://github.com/rust-embedded/itm) with a deprecation notice in favour of [`rtic-scope/itm`](https://github.com/rtic-scope/itm).

# Motivation
[motivation]: #motivation

`rust-embedded/itm` has been effectively superseded by the `rtic-scope/itm` fork.
This fork offers, in addition to the functionality of the previous implementation (v0.3), an `Iterator`-based design, more granular error enums, synchronization packet support, and timestamp generation of trace packets.
Planned future functionality at the time of writing is
- arbitration of instrumentation and extension packets such that payloads written on the target device via `cortex_m::iprint("message/payload")` (or similar API) can be trivially decoded by the library end-user; and
- an asynchronous API.

This fork also offers an `itm-decode` CLI tool that replaces `itmdump`.

[A pull-request replacing `rust-embedded/itm`](https://github.com/rust-embedded/itm/pull/41) with the fork under the umbrella of the Rust-Embedded Working Group (WG) was discussed during [a Matrix chat meeting (held 2021-11-31)](https://matrix.to/#/!BHcierreUuwCMxVqOf:matrix.org/$OcmpjhKy4iOk_5uQyhUpfVDA5_MtnNc1PkHVUDodSc8?via=matrix.org&via=psion.agg.io&via=beeper.com).
Merging this pull-request would require the WG to support a new code base and vet any future changes to its implementation and API.
As the library has yet to stabilize rapid changes can thus be expected which may cause release friction due to the vetting required by at least one WG member that is not proposing the changes themselves.
Questions whether host-side ITM software should be handled under the WG also arose.
A consensus regarding the merge was not reached.

A compromise is thus proposed:
archive `rust-embedded/itm` with a deprecation notice that points to the fork's repository at `https://github.com/rtic-scope/itm`, and give its maintainer (@tmplt) publish access to [the crates.io registery entry of `itm`](https://crates.io/crates/itm).
After publisher access has been granted, `itm v0.4.0` shall be releases that breaks compatibility with previous releases.
After the fork has stabilized discussion on whether to adopt the crate back into the WG again can resume.
The crate is unlikely to stabilize before Q1 2022.

# Unresolved questions
[unresolved]: #unresolved-questions

Whether the `itm` crate should ultimately be maintained by the WG.
