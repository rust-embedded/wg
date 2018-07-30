# Summary

This RFC proposes that we:

- Move crates that are widely used in the embedded ecosystem into the [rust-embedded] organization

- Significantly expand the membership of the rust-embedded org and organize the members into teams
  focused in different areas.

- Create guidelines for developing and maintaining the crates in the rust-embedded org.

- Settle on a policy for the inclusion of new crates and creation of new teams.

[rust-embedded]: https://github.com/rust-embedded

# Detailed design

## Teams

### Role

- The role of a team is to oversee the development of the crates under their purview. With input
  from stakeholders they set the development roadmap of each crate assigned to them.

- The members of a team are the main reviewers of the crates assigned to the team. Teams are
  free to add collaborators (reviewers) to the repositories they oversee, and to increase / modify
  their membership as they see fit.

- The team is in charge of publishing new versions of the crates they oversee to crates.io, and of
  maintaining the CHANGELOG of the project.

### Membership

- For visibility, the membership of each team, along with the set of crates they oversee, will be
  listed in the README of the rust-embedded/wg (previously known as rust-lang-nursery/embedded-wg)
  repository.

- To expand the membership of a team the candidate will send a PR to the rust-embedded/wg repository
  modifying the README to add themselves as a new member. The current members of the team will
  review the PR. If all the members approve the PR will be merged and the membership change will be
  made effective.

### The triage team

A special team named `triage` will be also be created. This team has the *very* important task of
keeping the PR queues moving and making sure no PR gets stuck in the review limbo.

- This team will consist of volunteers that will triage open PRs to rust-embedded repos on a weekly
  basis.

- The `triage` team can't review PRs; they can only change the labels of the PR and comment on the
  PRs.

- The `triage` team will follow the triage process used in [the rust-lang/rust repo]:
  - Open PRs will get assigned one of these status label: `S-waiting-on-author` or
  `S-waiting-on-reviewer`.
  - If the PR has no assigned reviewer assign a reviewer and set the `S-waiting-on-reviewer` label.
  - If around a week has passed since a reviewer was assigned ping the reviewer to remind them about
    the PR.
  - If the reviewer requested changes to the PR apply the `S-waiting-on-author` label.
  - If around a week has passed since changes were requested ping the author to remind them that
    changes are needed.
  - If the author made the requested changes change the label to `S-waiting-on-reviewer`.
  - If the author hasn't responded to pings for over two weeks, close the PR.

[the rust-lang/rust repo]: https://github.com/rust-lang/rust/pulls

## Crate inclusion policy

Only crates that are widely used and / or that have a wide scope will be maintained by the
embedded-rust org. Additionally, the crate must compile on stable Rust 1.31. In principle, this
excludes HAL implementations crate like the [`stm32f103xx-hal`] but exception can be made for crates
with narrow scope, like the [`f3`] crate, that are used by widely used resources like the
[`discovery`] book.

[`stm32f103xx-hal`]: https://github.com/japaric/stm32f103xx-hal

The procedure to add a crate is similar to the one used to add a new member to a team. The author
of the crate will make a PR modifying the README of the rust-embedded/wg repo. The team that
will adopt the crate will review the PR. If everyone accepts, the PR will be merged, the author will
transfer ownership of the repo to the org, and the author will become a collaborator.

## Reviewing guidelines

- The `master` branches will be protected and can't be directly pushed to.

- All changes will go through a PR, which must pass review and [bors] before landing. A team member
  / collaborator can't review / approve their own PR.

[bors]: https://bors.tech

- Breaking and major changes must be consulted with other team members, reviewers and stakeholders
  before the PR is reviewed. A mini [RFC] must be written listing the rationale of the change and
  the alternatives.

[RFC]: https://github.com/rust-lang/rfcs#table-of-contents

- Mandatory: Read the [Rust API guidelines] before you start reviewing PRs that add new API.

[Rust API guidelines]: https://rust-lang-nursery.github.io/api-guidelines/about.html

## Publishing guidelines

- Publish patch releases (bug fixes) often.

- Publish minor releases as soon as all the new features have been properly documented.

- Try to avoid *frequent* major (breaking) releases. Collect several breaking changes into a single
  major release.

- Before publishing a breaking release, read [semver-trick] and apply it to minimize breakage.

[semver-trick]: https://github.com/dtolnay/semver-trick

## Repository guidelines

- The README of all repositories will indicate which team is in charge of the repository and it will
  link to the teams sections of the README of the rust-embedded/wg repo.

- All the repositories that are crates will maintain a CHANGELOG.md file that adheres to the [keep a
  changelog] format.

[keep a changelog]: https://keepachangelog.com/en/1.0.0/

- The repository will include a copy of Rust [Code of Conduct]. Each team will enforce the code of
  conduct in the repositories they govern.

[Code of Conduct]: https://github.com/rust-lang/rust/blob/master/CODE_OF_CONDUCT.md

- Where appropriate the repositories will adopt the 2018 milestones used in the rust-embedded/wg
  repository.

## Other changes

- `rust-lang-nursery/embedded-wg` -> `rust-embedded/wg`
- The embedded Rust book and the embedonomicon which currently live in `rust-embedded/wg` will be
  moved into their own repositories.
- The newletters in `rust-embedded/wg` will also be moved into their own repository.
- The logo of the rust-embedded logo will change to the embedded WG logo.
- All members of the rust-embedded org will be able to manage issues and PRs in the rust-embedded/wg
  repository.

# Alternatives

An alternative to this proposal is to have the architectures teams be their own independent orgs.

# Unresolved questions

> Where is the AVR team?

Only architectures with built-in support, or that will soon have built-in support, in `rustc` have
been considered in this proposal. We can spin up more teams in the future.

- Should we require that would-be team members do some time as collaborators before they apply?

- The procedure for having a team member retire is still TBD.

- The procedure for removing (deprecating?) a crate from the rust-embedded org is still TBD.

- The procedure for creating a brand new team is still TBD.
