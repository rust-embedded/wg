- Feature Name: arm-team
- Start Date: 2025-02-11
- RFC PR: https://github.com/rust-embedded/wg/pulls/818

# Summary
[summary]: #summary

Merge the Cortex-A, Cortex-R and Cortex-M teams into a new Arm team, taking
over the previous repositories and team members.

# Motivation
[motivation]: #motivation

* Allow sharing maintenance of all the Arm crates including the Cortex-R crates
  which currently have no maintainers
* Better allow sharing of things that are common to all Arm architectures
* Reduce large number of wg teams

# Detailed design
[design]: #detailed-design

1. Create new Arm team
2. Add all Cortex-A, Cortex-R, and Cortex-M members to Arm team
3. Add all Cortex-A, Cortex-R, and Cortex-M repositories to Arm team
4. Delete Cortex-A, Cortex-R, and Cortex-M teams

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Update the WG README.

# Drawbacks
[drawbacks]: #drawbacks

* Some existing team members may not have any interest or experience in other Arm platforms

# Alternatives
[alternatives]: #alternatives

* Keep existing Cortex teams

# Unresolved questions
[unresolved]: #unresolved-questions

N/A
