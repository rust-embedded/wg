- Feature Name: Changing the way we handle addition/changes to the embedded-hal traits
- Start Date: 2019-11-12
- RFC PR: https://github.com/rust-embedded/wg/pull/415/
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

This proposal proposes to change the way how we make additions and changes to the `embedded-hal` traits in order to break the radio silence.

# Motivation
[motivation]: #motivation

Our current approach to changes is to open an RFC issue/PR, propose a trait, discuss it extensively and then maybe add it under the `unproven` feature flag -- or often not. In essence this means that all proposals are somewhat theoretical, reluctantly implemented and the `unproven` feature gates usually never get removed due to a lack of motivation to do that extra work which also leads to everyone uncoditionally enabling the `unproven` feature.

This is not very efficient for the community and stifles implementation and innovation.
 
This RFC attempts to change from a theoretical approach to a more real life approach by favouring working implementations over dry discussion.

# Detailed design
[design]: #detailed-design

This RFC suggests a more progressive approach by changing the way the crate works and the way we accept new additions and changes.

## Disabling the problematic `unproven` feature

This crate should only contain proven traits, hence all existing traits should be automatically considered proven (also to not break compatibility) and thus all feature gates removed and the feature itself turned into a no-op.

If a trait does not yet provide the expected quality, a new version can easily added under the new rules explained below.

## Accepting new additions

Instead of discussing a trait, adding it and hoping for implementations, all proposed changes should be required to demonstrate their usefulness by pointing to at least **two** independent and different implementations and at least one example utilizing the implementations. The implementations can live in separate branches or PRs of two or more repositories, are meant to demonstrate that the proposed traits can be implemented for arguably different hardware and are usable.

Based on these requirements a more meaningful and interactive discussion can take place, eventually resulting in acceptance of the aditions with working implementations already in hand.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

The new process would be documented accordingly so interested parties can easily follow it.

# Drawbacks
[drawbacks]: #drawbacks

None known.

# Alternatives
[alternatives]: #alternatives

Don't implement this RFC.

# Unresolved questions
[unresolved]: #unresolved-questions

* How many implementations do we require? Are two enough?
* Do we want to make a sufficient seperation of the mandatory implementations a hard requirement or a soft one? And if it's a hard requirement what would be the metrics for it?
