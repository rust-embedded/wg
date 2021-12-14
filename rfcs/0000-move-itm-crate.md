- Feature Name: move_itm_crate
- Start Date: 2021-12-01
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Disown from the Rust-Embedded Working Group (henceforth referred to as the "WG") and move the [`itm`](https://github.com/rust-embedded/itm) repo and [crates.io registery entry](https://crates.io/crates/itm) to @tmplt.

# Motivation
[motivation]: #motivation

The `itm` crate has been effectively superseeded by [`itm-decode`](https://crates.io/crates/itm-decode), specifially the [`rtic-scope:master`](https://github.com/rust-embedded/itm/pull/41) fork.
This fork offers, in addition to the functionality of the `itm` crate, an `Iterator`-based design, more granular error enums, synchronization packet support, and timestamp generation of trace packets.
Planned future functionality at the time of writing is
- arbitration of instrumentation and extension packets such that payloads written on the target device via `cortex_m::iprint("message/payload")` can be trivially decoded by the library end-user; and
- an asynchronous API.

This fork also offers an `itm-decode` CLI tool that replaces `itmdump`.

The pull-request replacing `itm` with the fork under the umbrella of the WG was discussed during  [yesterday's (2021-11-31) Matrix chat meeting](https://matrix.to/#/!BHcierreUuwCMxVqOf:matrix.org/$OcmpjhKy4iOk_5uQyhUpfVDA5_MtnNc1PkHVUDodSc8?via=matrix.org&via=psion.agg.io&via=beeper.com).
Merging this pull-request would require the WG to support a new code base and vet any future changes to its implementation and API.
As the library has yet to stabilize rapid changes can thus be expected which may cause release friction due to the vetting required by at least one WG member that is not proposing the changes themselves.
Questions whether host-side ITM software should be handled under the WG also arose.
A consensus regarding the merge was not reached.

An inbetween approach is thus proposed:
disown from the WG and move ownership of the `itm` repo and crate to @tmplt such that development can continue without friction, and so that there is one canonical ITM decoding crate to avoid end-user confusion.
After the pull-request discussed above have been merged and the crate has stabilized discussion on whether to adopt the crate into the WG again can resume.
The crate is unlikely to stabilize before Q1 2022.

# Unresolved questions
[unresolved]: #unresolved-questions

Whether the `itm` crate should ultimately be maintained by the WG.
