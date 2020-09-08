- Feature Name: rust-embedded-community as child organization
- Start Date: 2020-09-07
- RFC PR:
- Rust Issue:

# Summary
[summary]: #summary

Adopt the [Rust Embedded Community (REC)][rec] as an official child organization/subproject of the Rust Embedded Working Group (REWG).

# Motivation
[motivation]: #motivation

## Maturity segmentation

Foundational libraries and tooling for Rust usage in embedded context are part of the official REWG organization. As such, these libraries can be seen as especially well maintained or thought-through within the broad embedded rust community.

It would be interesting to have a "playground" for less mature libraries where its status is clear to users while at the same time centralizing efforts.

Users have already shown interest in something like this. See: [rust-embedded-community/meta#5][rec-5], [rust-embedded-community/meta#6][rec-6]

## Home for unmaintained projects

There are libraries that were once developed but the original author does not have the time necessary for further development or has moved on to other projects.

The REC provides a place for these libraries to live on, offering the opportunity for development to be easily aided by or taken over by new members. e.g. Bus factor increase.

## Extended membership

A separate organization poses a "less serious" opportunity to gather new members. Engagement may result in future REWG membership.

## Why a child organization

The REWG's mission is to improve the experience of using Rust in embedded contexts. Centralization of community projects and maintenance thereof inside the REC organization also helps here.

Projects being in a different organization does not mislead the user into thinking that a particular library is on the same maturity/maintenance level as other libraries in the REWG.

Furthermore, for select experimental libraries, a child organization provides a clear evolutionary path for centralized refinement with the ultimate goal of adoption by the REWG or REWG maintainers once they achieve sufficient level of maturity if these are deemed foundational-enough.

The name "Rust Embedded Community" could be confused with the "Rust Embedded Working Group". On the other hand, it seems difficult to find a different but still purposeful name for such a general organization. A child organization allows for use of this name and thus reduces fragmentation in the ecosystem.

# Detailed design
[design]: #detailed-design

The [REC] already exists and hosts a number of libraries.

## Governance by the REWG

In order to protect the interests of the broad embedded Rust community, a group of REWG members will form a governance team within the REC.

This team's tasks would be:
- Ensure civility and ultimately enforce the code of conduct.
- Ultimate decisions regarding member/team permissions. e.g. crate publication rights management.
- Protect Rust logo trademark.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Link to REC from this repository as well as on awesome-embedded-rust.

# Drawbacks
[drawbacks]: #drawbacks

- Unmaintained libraries piling up may look bad.
- Maintenance burden.
- [Current logo proposal][logo] may seem too similar to REWG's.

# Alternatives
[alternatives]: #alternatives

- Keep the REC independent
    - The REC needs a new name if the REWG thinks it is too similar.
    - Centralization of development happens outside of the REWG.

# Unresolved questions
[unresolved]: #unresolved-questions

[Current REC logo proposal][logo] based on the REWG logo must be resubmitted for consideration of Rust's core team under these new terms.

[logo]: https://user-images.githubusercontent.com/43125/88387805-8f957c00-cdb3-11ea-8b08-0259c6efbd74.png
[rec]: https://github.com/rust-embedded-community
[rec-5]: https://github.com/rust-embedded-community/meta/issues/5
[rec-6]: https://github.com/rust-embedded-community/meta/issues/6
