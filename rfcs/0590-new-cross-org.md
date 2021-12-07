- Feature Name: new_cross_org
- Start Date: 2021-12-06
- RFC PR: [#590](https://github.com/rust-embedded/wg/pull/590)

# Summary
[summary]: #summary

Move the `cross` repository to a new GitHub organisation `cross-rs`,
with volunteers from the current Tools team as initial owners of the new
organisation, to continue development of `cross` outside of the working gruop.

# Motivation
[motivation]: #motivation

Currently the working group is responsible for maintaining
[`cross`](https://github.com/rust-embedded/cross), a popular tool for
cross-compiling Rust projects to different targets. While it was initially also
useful for embedded development, it's no longer especially useful for standard
embedded use (since Rust natively supports many popular targets), but has
become popular for other use cases.

Because it's no longer widely used for embedded, it has not received much
maintenance from the embedded WG, with the last release over a year ago
and a number of issues and PRs left open.

We've previously asked for help with new maintainers [in this
thread](https://github.com/rust-embedded/cross/issues/574) and received a
positive response, but it doesn't seem appropriate to require these new
maintainers to join an existing WG team and no action has been taken to
bring any of the volunteers into the organisation.

Essentially, the WG does not have the maintenance bandwidth to keep on top
of `cross`, but its user community hopefully does.

# Detailed design
[design]: #detailed-design

We transfer the `cross` repository to a new `cross-rs` organisation (created in
advance of writing this RFC to ensure name availability). Volunteers from the
existing Tools team will be added as owners to the new organisation, which is
otherwise a separate entity from the working group. In addition, some
volunteers from the call-for-help issue could be added as maintainers and
eventually organisation owners. A new GitHub team will be created and granted
publish rights to crates.io.

The new organisation and moved repository would continue the WG's commitment to
upholding the Rust project code of conduct, and the existing Tools team members
would provide continuity of ownership and oversight as the new project gets
underway.

Currently cross uses Azure Pipelines for CI and the dockerhub container
registry for container images, but it is envisioned this will swap to GHA
and GHCR, which means CI allowances and authorisation will be provided through
the new GitHub organisation. The current status of the Azure authentication
details are unknown, and the dockerhub team is beyond the free limit, so
continuing with those providers is not anticipated.

# Drawbacks
[drawbacks]: #drawbacks

Current users of cross rely on the WG to provide a reliable, stable, and
trustworthy tool, and abandoning it to a new organisation could betray that
trust. However, the maintenance level today is insufficient to provide the
desired tool quality and unlikely to change significantly in the near future.
By populating the new organisation initially with the existing maintainers,
continuity of ownership is provided.

# Alternatives
[alternatives]: #alternatives

Instead of creating a new organisation, we could:

* Create a new team inside the working group and have that team manage cross,
  adding new volunteers to the new team. However, this means the new volunteers
  who are only interested in cross must become WG members too, and the WG must
  continue to maintain a tool that is no longer especially relevant to
  embedded.
* Continue with just the Tools team maintaining cross, potentially adding new
  volunteers to the Tools team. This is pretty much the status quo, so does
  not resolve the current maintenance problems.

# Unresolved questions
[unresolved]: #unresolved-questions

None at this time.
