# Publishing workflow

## Pull request

Open a PR that:

- Updates `CHANGELOG.md` to include the new version. We use the [Keep a Changelog] format. Don't
  forget to include the version link at the bottom. Example below:

[Keep a Changelog]: https://keepachangelog.com/en/1.0.0/

``` markdown
## [Unreleased]

<!-- TODO Move everything in this section to the new version section below -->

<!-- TODO Add this new section -->
## [v0.5.3] - 2018-08-02

### Added

- Some stuff.

### Fixed

- Some stuff.

### Removed

- Some stuff.

<!-- NOTE This was already here -->
## [v0.5.2] - 2018-05-18

<!-- ... omitting stuff in between ... -->

<!-- TODO Change the start of the range -->
[Unreleased]: https://github.com/rust-embedded/cortex-m/compare/v0.5.3...HEAD

<!-- TODO Add this new link -->
[v0.5.3]: https://github.com/rust-embedded/cortex-m/compare/v0.5.2...v0.5.3

<!-- NOTE This was already here -->
[v0.5.2]: https://github.com/rust-embedded/cortex-m/compare/v0.5.1...v0.5.2
```

- Bumps the crate version in `Cargo.toml`.

## `cargo publish`

After the PR has been merged, run `cargo publish` locally

## Tag

Afterwards, create an annotated tag and push it

``` console
$ git tag -a 'v0.5.3' -m 'v0.5.3'

$ git push origin v0.5.3
```
