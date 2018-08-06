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

This section describes the role of the teams and mechanisms for changing their memberships.

### Role

The role of a team is to oversee the development of the crates under their purview. With input from
stakeholders they set the development roadmap of each crate assigned to them.

The members of a team are the main reviewers of the crates assigned to the team. Teams are free to
add collaborators (reviewers) to the repositories they oversee, and to increase / modify their
membership as they see fit.

The team is in charge of publishing new versions of the crates they oversee to crates.io, and of
maintaining the CHANGELOG of the project.

### Membership

For visibility, the membership of each team, along with the set of crates they oversee, will be
listed in the README of the rust-embedded/wg (previously known as rust-lang-nursery/embedded-wg)
repository.

#### Adding new members

A person that wants to join a team (the "candidate") will first consult with (part of) the team. The
team may decide that the candidate participates as a collaborator to one of the repositories
maintained by the team before they apply for membership. After receiving a soft OK from (part of)
the team the candidate will send a PR to the rust-embedded/wg repository modifying the README file
to list themselves as a new member.

The complete team will review the PR and perform one of these 3 actions: approve the PR, abstain
from reviewing, or raise a concern. When the PR achieves the *majority* of votes (defined below) it
will be merged and the membership changes will be made effective.

#### Voting majority

Voting majority is defined as having more approvals than abstentions and zero unresolved concerns.
In the case where there are equal numbers of approvals and abstentions (i.e. the team has an even
number of members) the WG lead will give the casting vote.

#### Removing members

When (part of) a team wishes to remove one of its member they will make a PR to the
rust-embedded/wg repository updating the README with the modified membership. The team minus the
member being removed will vote on the proposal. After voting majority (defined in the previous
section) has been achieved the PR will be merged and the membership changes will be made effective.

In the case where the member wishes to retire the procedure is similar. The retiring member will
make a PR notifying the team (e.g. "cc rust-embedded/$team"). In this case no voting is necessary.
The PR will be merged and the changes will be made effective.

### The triage team

A special team named `triage` will be also be created. This team has the *very* important task of
keeping the PR queues moving and making sure no PR gets stuck in the review limbo.

This team will consist of volunteers that will periodically triage open PRs to rust-embedded repos.
When triaging the members of the `triage` team will assign or change reviewers, add or change the
labels of the PR, and/or comment on the PRs; they will not review PRs while triaging.

#### The triage procedure

The proposed triage procedure is [the one used to triage PRs in the rust-lang/rust
repo][rust-triage].

[rust-triage]: https://forge.rust-lang.org/triage-procedure.html

## Projects

The section describes the guideline for incorporating projects into the rust-embedded organization
and the mechanism for adopting projects.

### Inclusion guideline

Only projects (crates or documentation) that are widely used and / or that have a wide scope will be
maintained by the embedded-rust org. Additionally, if the project is a crate it must compile on the
stable channel by Rust 1.31 (2018-12-06); if the project is documentation all the included code and
patterns must be usable on the stable channel.

In principle, this guideline excludes HAL implementation crates like the [`stm32f103xx-hal`] and
board support crates because of their narrow scope but exceptions could be made for crates, like the
[`f3`] crate, that are dependencies of widely used resources like the [`discovery`] book.

[`stm32f103xx-hal`]: https://github.com/japaric/stm32f103xx-hal
[`f3`]: https://github.com/japaric/f3
[`discovery`]: https://github.com/japaric/discovery

### Adopting projects

An author that wishes to have their project adopted by the rust-embedded org should first approach
the team that would oversee their crate if it were to be adopted. If the author is unsure which team
should take ownership of their project they will first open an issue in the rust-embedded/wg
repository requesting input.

After getting a soft OK from (part of) the team the author will send a PR to rust-embedded/wg adding
their repository to the list of crates the team maintains. After voting majority (see definition in
the membership section) the PR will be merged, ownership will be transferred and the author will
become a collaborator of the transferred repository.

## Guidelines

This section contains a set of guidelines that aim to standardize the management of projects
within the rust-embedded org.

### Reviewing

- The `master` branches will be protected and can't be directly pushed to.

- All changes will go through a PR that must pass review and [bors] before landing. Team members
  and collaborators can't approve their own PRs.

[bors]: https://bors.tech

- Breaking and major changes must be consulted with other team members, reviewers and stakeholders
  before the PR is reviewed. The team will decide whether a mini [RFC], listing the rationale of the
  change and the alternatives, needs to be written.

[RFC]: https://github.com/rust-lang/rfcs#table-of-contents

- Mandatory: Read the [Rust API guidelines] before you start reviewing PRs that add new API.

[Rust API guidelines]: https://rust-lang-nursery.github.io/api-guidelines/about.html

### Publishing

- Publish patch releases (bug fixes) often.

- Publish minor releases as soon as all the new features have been properly documented.

- Try to avoid *frequent* major (breaking) releases. Collect several breaking changes into a single
  major release.

- Before publishing a breaking release, read [semver-trick] and apply it to minimize breakage.

[semver-trick]: https://github.com/dtolnay/semver-trick

### Repository contents

- The README of all repositories will indicate which team is in charge of the repository and it will
  link to the teams section of the README in the rust-embedded/wg repo.

- All the repositories that are crates will maintain a CHANGELOG.md file that adheres to the [keep a
  changelog] format.

[keep a changelog]: https://keepachangelog.com/en/1.0.0/

- The repository will include a copy of Rust's [Code of Conduct]. Each team will enforce the code of
  conduct in the repositories they govern.

[Code of Conduct]: https://github.com/rust-lang/rust/blob/master/CODE_OF_CONDUCT.md

- Where appropriate the repositories will adopt the 2018 milestones used in the rust-embedded/wg
  repository.

## Other changes

- Transfer the `rust-lang-nursery/embedded-wg` to the rust-embedded org and rename it to
  `rust-embedded/wg`.

- The embedded Rust book and the embedonomicon which currently live in `rust-embedded/wg` will be
  moved into their own repositories.

- The newletters in `rust-embedded/wg` will also be moved into their own repository.

- The logo of the rust-embedded org will change to the embedded WG logo.

- All members of the rust-embedded org will be able to manage issues and PRs in the rust-embedded/wg
  repository.

# Alternatives

An alternative to this proposal is to have the architectures teams be their own independent orgs.
