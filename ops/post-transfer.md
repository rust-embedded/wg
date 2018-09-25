# Post transfer TODO list

## Travis CI

Browse to https://travis-ci.org/rust-embedded/repo-name/settings and make sure that "Build pushed
pull requests" is enabled.


## Source

Push a commit to `master` that does the following:

- Creates `.github/CODEOWNERS` with the following contents:

``` text
* rust-embedded/team-name collaborator-name another-collaborator
```

- Adds a copy of [`CODE_OF_CONDUCT.md`][CoC]. Don't forget to adjust the team name and associated
  link accordingly.

[CoC]: https://github.com/rust-embedded/cortex-m/blob/master/CODE_OF_CONDUCT.md

- Modifies `.travis.yml` to:
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

- If the repository uses CI, configure bors. Add a `.github/bors.toml` file with these contents:

``` toml
block_labels = ["needs-decision"]
delete_merged_branches = true
required_approvals = 1
status = ["continuous-integration/travis-ci/push"]
```

## Bors

If the repository uses CI, update bors settings.

- Browse to https://app.bors.tech/repositories, click on rust-embedded/repo-name and then switch
  to the "Settings" tab.

- Synchronize both "Reviewers" and "Members" to people with "Push" (write) access to the
    repository.

## Repository

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
