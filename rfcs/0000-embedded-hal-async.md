- Feature Name: Take over embedded-hal-async
- Start Date: 2021-12-20
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

The HAL team of the Rust Embedded WG takes over the crate `embedded-hal-async`.

# Motivation
[motivation]: #motivation

The `embedded-hal-async` crate has been created to serve as a central place for developing
asynchronous versions of the `embedded-hal` traits.
Given the interest from many members of the community, it would be useful to pursue
this in a centralized fashion like `embedded-hal`.

# Detailed design
[design]: #detailed-design

@eldruin performs whichever small adaptions necessary and transfers the repository
to the rust-embedded organization.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

We add a link to the `embedded-hal-async` repository in `embedded-hal` and in its
GitHub issues which refer to asynchronous matters.

# Drawbacks
[drawbacks]: #drawbacks

Additional maintenance burden.

# Alternatives
[alternatives]: #alternatives

1. Keep this crate outside of the WG.
2. Split the `embedded-hal` repository in-place: https://github.com/rust-embedded/embedded-hal/pull/334

# Unresolved questions
[unresolved]: #unresolved-questions

None.

[`embedded-hal-async`]: https://github.com/eldruin/embedded-hal-async