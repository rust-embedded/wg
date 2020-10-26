# Post transfer TODO list

## Repository changes

- Mention the Code of Conduct and which team owns this repository in the README

``` markdown
# project-name

> description of the project

<!-- TODO add this -->

This project is developed and maintained by the [Cortex-M team][team].

<!-- ... omitting stuff in between ... -->

<!-- TODO add this -->

## Code of Conduct

Contribution to this crate is organized under the terms of the [Rust Code of
Conduct][CoC], the maintainer of this crate, the [Cortex-M team][team], promises
to intervene to uphold that code of conduct.

[CoC]: CODE_OF_CONDUCT.md
[team]: https://github.com/rust-embedded/wg#the-cortex-m-team
```

- Add a copy of [`CODE_OF_CONDUCT.md`][CoC]. Don't forget to adjust the team name and associated
  link accordingly.

[CoC]: https://github.com/rust-embedded/cortex-m/blob/master/CODE_OF_CONDUCT.md

- Create `.github/CODEOWNERS` with the following contents:

``` text
* rust-embedded/team-name collaborator-name another-collaborator
```

## CI via Travis CI

### Changes in Travis CI

Browse to https://travis-ci.org/rust-embedded/repo-name/settings and make sure that "Build pushed
pull requests" is enabled.

### Changes in the repository

Push a commit to `master` that does the following:

- Modify `.travis.yml` to:
  - Allow builds on these three branches. `staging` and `trying` are required by bors. We include
    `master` to have builds on pushed PRs.

``` yaml
branches:
  only:
    - master
    - staging
    - trying
```

  - Prevent builds on changes to master. This is to prevent testing a PR *twice*: one build is
    required to land the PR, but when the changes are merged into master we want to avoid that
    second build.

``` yaml
matrix:
  include:
    - env: TARGET=thumbv6m-none-eabi
      # add this `if` ...
      if: (branch = staging OR branch = trying) OR (type = pull_request AND branch = master)

    - env: TARGET=thumbv7m-none-eabi
      # ... to all the elements of the build matrix
      if: (branch = staging OR branch = trying) OR (type = pull_request AND branch = master)
```

- Add a `.github/bors.toml` file with these contents:

``` toml
block_labels = ["needs-decision"]
delete_merged_branches = true
required_approvals = 1
status = ["continuous-integration/travis-ci/push"]
```
## CI via GitHub Actions (GHA)

### Changes to GitHub repository configuration

No changes are needed as GHA is automatically enabled by default for every organisation.

### Changes in the repository

Push a commit to `master` that does the following:

#### Add a `.github/workflows/rustfmt.yml`

  The purpose of this workflow is to ensure proper code formatting, it should
  always be the same and look like:

``` yaml
on:
  push:
    branches: [ staging, trying, master ]
  pull_request:

name: Code formatting check

jobs:
  fmt:
    name: Rustfmt
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions-rs/toolchain@v1
        with:
          profile: minimal
          toolchain: stable
          override: true
          components: rustfmt
      - uses: actions-rs/cargo@v1
        with:
          command: fmt
          args: --all -- --check
```

#### Add a `.github/workflows/clippy.yml`

  The purpose of this workflow is to ensure following the Rust coding best
  practices. It should always be the same and look like:

``` yaml
on:
  push:
    branches: [ staging, trying, master ]
  pull_request:

name: Clippy check
jobs:
  clippy_check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions-rs/toolchain@v1
        with:
          profile: minimal
          toolchain: stable
          override: true
          components: clippy
      - uses: actions-rs/clippy-check@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

#### Add a `.github/workflows/ci.yml`

  This contains the "meat" of the CI by testing the compilability of the code in
  a specific setting, potentially also run unit- or doctests and ensure the MSRV
  and future compatibility. Thus it will always be repository specific but we'll
  give a few hints here what to look out for:

  - Make sure to include these lines as preamble to ensure that `bors` will
    function properly and PRs are not unnecessarily tested multiple times per change:
``` yaml
on:
  push:
    branches: [ staging, trying, master ]
  pull_request:

name: Continuous integration
```

- Define your jobs according to the host system(s) you'd like to run on and all
  the target variants which need testing.
  The snippet is set up to:
  - only test on linux machines, hence the job name `ci-linux`
  - uses the latest ubuntu
  - setup a matrix which runs all tests for stable Rust and deviates for others
  - Defines a `TARGET` env variable containing all default targes
  - Defines deviations for the MSRV test (using Rust `1.35.0` and only tests for `x86_64-unknown-linux-gnu`)
  - Defines deviations for nightly tests and marks them as `experimental`, i.e.
    it will only generate warnings if this part fails instead of aborting the whole job
``` yaml
jobs:
  ci-linux:
    runs-on: ubuntu-latest
    continue-on-error: ${{ matrix.experimental || false }}
    strategy:
      matrix:
        # All generated code should be running on stable now
        rust: [stable]

        # The default target we're compiling on and for
        TARGET: [x86_64-unknown-linux-gnu, thumbv6m-none-eabi, thumbv7m-none-eabi]

        include:
          # Test MSRV
          - rust: 1.35.0
            TARGET: x86_64-unknown-linux-gnu

          # Test nightly but don't fail
          - rust: nightly
            experimental: true
            TARGET: x86_64-unknown-linux-gnu
```

- Then we need to add the build steps for GHA to execute (for each multiplied
  out element from the matrix). For GHA we have a nice set of predefined actions
  which we can utilize to simplify our lives, this might typically look like:

``` yaml
    steps:
      - uses: actions/checkout@v2
      - uses: actions-rs/toolchain@v1
        with:
          profile: minimal
          toolchain: ${{ matrix.rust }}
          target: ${{ matrix.TARGET }}
          override: true
      - uses: actions-rs/cargo@v1
        with:
          command: check
          args: --target=${{ matrix.TARGET }}
```


#### Add a `.github/bors.toml` file with these contents:

The status checks are unfortunately a bit tedious as wildcard on the status can
**not** be used here, so the information needs to be customized to match the
information in the `ci.yml`, e.g. for the above snippets the required
`bors.toml` would look like:

``` toml
block_labels = ["needs-decision"]
delete_merged_branches = true
required_approvals = 1
status = [
  "ci-linux (stable, x86_64-unknown-linux-gnu)",
  "ci-linux (stable, thumbv6m-none-eabi)",
  "ci-linux (stable, thumbv7m-none-eabi)",
  "ci-linux (1.35.0, x86_64-unknown-linux-gnu)",
]
```

## Bors

If the repository uses CI, update bors settings.

- Browse to https://app.bors.tech/repositories, click on rust-embedded/repo-name and then switch
  to the "Settings" tab.

- Synchronize both "Reviewers" and "Members" to people with "Push" (write) access to the
    repository.

## Highfive

Highfive is a service which automatically selects a suitable reviewer for a pull request. In order
to do so it needs to be instructed about the reviewers via configuration in the Highfive repository
https://github.com/rust-lang/highfive.

A suitable configuration file per crate will be located under
https://github.com/rust-lang/highfive/tree/master/highfive/configs/rust-embedded and looks like

```
{
    "groups": {
        "all": ["rust-embedded/cortex-m"]
    },
    "new_pr_labels": ["S-waiting-on-review", "T-cortex-m"]
}
```

with `cortex-m` substituted by the responsible team.

## Repository settings

Update the repository settings.

- Browse to https://github.com/rust-embedded/repo-name/settings and switch to the "Collaborators &
  teams" tab.

- Add the team that will oversee the project and give them "Admin" permissions.

- The previous owner and old collaborators should have "Write" permissions.

- Now switch to the "Branches" tab and add a branch protection rule for `master` with the
  following settings:

  - [x] Require pull requests reviews before merging
    - [x] Dismiss stale pull request approvals when new commits are pushed
    - [x] Require review from Code Owners

  - [x] Require status checks to pass before merging (NOTE omit if not using CI)
    - [x] bors (NOTE if bors doesn't appear here you will have to make a dummy PR, `bors r+` it
      and then close it. This will make bors visible in this list)

  - [x] Include administrators

## crates.io

If applicable, add new owners to crates.io. Have the current owner add the team that's receiving
the crate as a team owner.

```
$ cargo owner --add github:rust-embedded:team-name
```
