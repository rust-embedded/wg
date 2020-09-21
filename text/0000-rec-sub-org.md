- Feature Name: rust-embedded-community as child organization
- Start Date: 2020-09-07
- RFC PR:
- Rust Issue:

# Summary
[summary]: #summary

Adopt the [Rust Embedded Community (REC)][rec] as an official child organization/subproject of the Rust Embedded Working Group (REWG).

# Motivation
[motivation]: #motivation

## Maturity/Applicability segmentation

Foundational libraries and tooling for Rust usage in embedded context are part of the official REWG organization. As such, these libraries can be seen as especially well maintained or thought-through within the broad embedded Rust community.

It would be interesting to have an additional segment dedicated to libraries that are too immature or too niche for the REWG. A separate organization makes it clear to users that these libraries are not on the same level as the ones at the REWG while at the same time offering a space for effort centralization.

Users have already shown interest in something like this. See: [rust-embedded-community/meta#5][rec-5], [rust-embedded-community/meta#6][rec-6]

## Home for unmaintained projects

There are libraries that were once developed but the original author does not have the time necessary for further development or has moved on to other projects.

The REC provides a place for those of these libraries which are accepted to live on, offering the opportunity for development to be easily aided by or taken over by a group of interested people, also to increase the "bus factor".

## Extended membership

A separate organization poses a "less serious" opportunity to gather new members. Engagement may result in future REWG membership.

## Why a child organization

The REWG's mission is to improve the experience of using Rust in embedded contexts. Centralization of community projects and maintenance thereof inside the REC organization also helps here.

Projects being in a different organization does not mislead the user into thinking that a particular library is on the same maturity/maintenance level as other libraries in the REWG.

Furthermore, for select experimental libraries, a child organization provides a clear evolutionary path for centralized refinement with the ultimate goal of adoption by the REWG or REWG maintainers once they achieve sufficient level of maturity if these are deemed foundational-enough (see transfer process below).
However, this only affects a small portion of libraries and it is _not_ the sole or main purpose of the REC to serve as an incubator for the REWG.

The name "Rust Embedded Community" could be confused with the "Rust Embedded Working Group". On the other hand, it seems difficult to find a different but still purposeful name for such a general organization. A child organization allows for use of this name and thus reduces fragmentation in the ecosystem.

# Detailed design
[design]: #detailed-design

The [REC] already exists and hosts a number of libraries.

## Governance by the REWG

In order to protect the interests of the broad embedded Rust community, a group of REWG members will form a governance team within the REC.

This team's tasks would be:
- Decide on new library adoptions.
- Decide on library transfer to the REWG.
- Ensure civility and ultimately enforce the code of conduct.
- Ultimate decisions regarding member/team permissions. e.g. crate publication rights management.
- Protect the Rust logo trademark.

## Adopting a new library

For a library to be adopted by the REC a PR to its [meta][rec-meta] repository must be made detailing the motivation for adoption.
Then, an approval by the REC governance team as described in [RFC-0206] based on the following criteria is necessary:

1. It is (primarily) written in Rust.
2. Has an [OSI]-approved open-source license.
3. It targets an embedded context.
4. It is useful for the broad embedded Rust community. 

In order to reduce efforts, an informal agreement with REC governance team members should be sought before starting this process.

## Transferring a project from the REC to the REWG

Whenever a library inside the REC is deemed of interest for the REWG, a transfer to the REWG is possible.
The requirements are:

1. An approval by the REC community (or team responsible for the library if it exists) including the REC governance team by means of PR to the REC's [meta][rec-meta] repository. See [RFC-0206] for reference.
2. An approval of the adoption by the REWG following REWG's own terms (See: [RFC-0136]).

In order to reduce efforts, an informal agreement with relevant REC members as well as with relevant REWG members should be sought before starting this process.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Link to REC from this repository as well as on awesome-embedded-rust clearly stating what the purpose of the REC is.

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
[rec-meta]: https://github.com/rust-embedded-community/meta
[OSI]: https://en.wikipedia.org/wiki/Open_Source_Initiative
[rec]: https://github.com/rust-embedded-community
[rec-5]: https://github.com/rust-embedded-community/meta/issues/5
[rec-6]: https://github.com/rust-embedded-community/meta/issues/6
[RFC-0136]: /rfcs/0136-teams.md
[RFC-0206]: /rfcs/0206-voting-majority.md
