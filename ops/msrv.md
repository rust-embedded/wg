# Minimum Supported Rust Version (MSRV)

This text documents the MSRV policy used in the crates maintained by the WG.

1. Crates released by the Embedded WG must compile on the most recent stable
   Rust release at all times. If a dependency releases an update which causes a
   published crate to no longer build on the most recent stable release, a new
   version must be released to resolve the issue.
2. Individual crates may specify a more restrictive MSRV if the crate's team
   agrees to do so, as long as it is at least as restrictive as this policy.
3. It is permissible for specifically-indicated features of a crate to not
   build on stable, to support the use of nightly-only features. All features
   not specifically indicated in the README or documentation must build on
   stable.

It is recommended that all crates use a CI system to check that a PR does not
break building on stable Rust, and schedule regular CI jobs to check that a
newly released stable Rust has not broken the crate's build. Crates may also
consider regular CI runs against the latest released version of the crate.

This policy was most recently updated by [RFC 0523].

[RFC 0523]: https://github.com/rust-embedded/wg/pull/523
