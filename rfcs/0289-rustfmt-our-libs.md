# Summary
[summary]: #summary

This RFC proposes to use CI to give errors, or bors to apply rustfmt on merge to
make sure our lib do not diverge in style within the WG's libraries. Moreover,
it can act as a reference for newcomers to also follow the same style.

# Motivation
[motivation]: #motivation

The main motivation is to keep the WG's style consistent. As libraries continue
to evolve over time, as will the style if we do not have a system to keep us in
check. It could also be used as an example to motivate the newcomers of embedded
Rust to use `rustfmt` and keep the style consistent. It helps everyone when
opening someones code and feeling at home, and not needing to reread the code to
understand the style of the writer, which is a common issue in for example C++.

# Detailed design
[design]: #detailed-design

All in all, two main methods were discussed:

1. Enforce `rustfmt` at CI level, and have an extra target for formating which
will fail if the code is not following the correct style. This does however mean
that there can be a lot of commit noise to just "fix CI style errors".

2. Have bors apply rustfmt on merge. This would be the painless method, but
would not act as an example for newcomers.

# Unresolved Questions
[unresolved]: #unresolved

* Should we do it?
* If so, which method to use?
