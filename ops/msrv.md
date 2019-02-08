# Minimum Supported Rust Version (MSRV)

This text documents the MSRV policy used in the crates maintained by the WG.

- The MSRV shall always be strictly smaller than the latest available minor
  stable release. For example, if the latest stable Rust version was 1.32.1 a
  new minor release of the `cortex-m` crate can't bump the MSRV higher than
  1.31.

- Changing the MSRV of a crate is a breaking change and requires a semver bump:
  minor version bump if the crate is pre-1.0 and a major version bump if the
  crate is (post-)1.0.

- Cargo features are allowed to depend a on Rust version greater than the MSRV,
  even a nightly compiler. For example, a "const-fn" can bump the required
  version to 1.34.0 (effectively, nightly channel at the time of writing).

- When doing a semver bump (i.e. minor version bump in pre-1.0 crates), for
  whatever reason, the team in charge of the crate should consider bumping the
  MSRV as well.

- The MSRV will be documented in the crate level documentation of the crate,
  like so:

``` markdown
# Minimum Supported Rust Version (MSRV)

This crate is guaranteed to compile on stable Rust 1.31 and up. It *might*
compile with older versions but that may change in any new patch release.
```

- The MSRV will be tested in the CI of the crate as a separate build job, like
  so:

``` yaml
matrix:
  include:
      # MSRV
    - env: TARGET=thumbv7m-none-eabi
      rust: 1.31.0

    - env: TARGET=thumbv7m-none-eabi
      rust: stable

    - env: TARGET=thumbv7m-none-eabi
      rust: beta

      # etc.
```
