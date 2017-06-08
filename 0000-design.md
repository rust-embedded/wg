- Feature Name: embedded design
- Start Date: 2017-06-08
- RFC PR: (leave this empty)

# Summary
[summary]: #summary

There should be design document/s to detail the different layers that we have
and want for the rust embedded ecosystem. These design documents should detail:
- the requirements: what we want and the general layout
- the specifications: the approach for each layer and possible APIs for
  libraries that might support it, including links to libraries that already
  exist/are under development
- testing: tools to help support in the testing of embedded programs.

# Motivation
[motivation]: #motivation

The embedded development ecosystem of rust is in the very earliest stages of
development, with new development in libraries, tooling and features happening
every day. In order to help give contributors direction on what is needed in the
ecosystem and to help tie the APIs of the ecosystem together it would be
beneficial to have design documents detailing the different library layers and
what libraries rust-embedded is hoping to support.

# Detailed design
[design]: #detailed-design

We should use the tool [artifact](https://github.com/vitiral/artifact)
to write design documents and host them.

I have already gotten a MVP of the design docs here:
https://vitiral.github.io/rust-embedded-design/#artifacts/REQ-PURPOSE

# Drawbacks
[drawbacks]: #drawbacks

Some members of the community may feel that we are trying to dictate how they
design their libraries. The goal should always be to try and work with
developers and make sure it is clear that we intend to support them.

It will take time to create and update a quality design of the ecosystem.
That time could be spent on writing the libraries. However, without good design
how do we know we are writing the *correct* libraries?

# Alternatives
[alternatives]: #alternatives

- We could not design at all.
- We could use a tool other than artifact

# Unresolved questions
[unresolved]: #unresolved-questions

There are plenty of unresolved questions -- such as what the design will be.
