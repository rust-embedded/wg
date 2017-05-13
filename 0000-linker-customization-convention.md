- Feature Name: linker_customization_convention
- Start Date: 2017-05-13
- RFC PR: (leave this empty)

# Summary

Set a convention for passing custom arguments to the linker when using Cargo for
embedded development. This RFC proposes adopting a project local configuration
file, `.cargo/config`, to pass the arguments.

# Motivation

Embedded development requires precise control over the memory layout of the
final binary artifact; this is usually achieved through a mixture of extra
linker arguments, object files and linker scripts. There is currently no
convention about how to pass extra arguments to the linker using Cargo / rustc
but there are currently two approaches used in practice:

- Write a custom target specification file, usually basing it on a built-in
  target, and then adding to it the linker arguments in the `{pre,,}-link-args`
  fields.

- Use a project local configuration file, `.cargo/config`, to pass linker
  arguments through the `target.$T.rustflags` variable.

A third alternative would be to add features to Cargo to let it control the
linking process through build scripts. This seems unlikely to be implemented in
the short term (or ever)
as [the downsides may outweigh the benefits][rustc-link-arg].

[rustc-link-arg]: https://github.com/rust-lang/rfcs/issues/1766#issuecomment-252034343

The goal is to RFC is to compare the advantages and disadvantages of each
approach and to settle on one of them as a convention.

# Rationale

Or, why `.cargo/config` should be preferred over custom targets for this
purpose:

Linker arguments, linker scripts and object files are usually device, board or
application specific. IMO, this maps nicely to `.cargo/config` as it's a project
local, and thus an application specific, configuration. OTOH, targets are more
global and are meant to be used across projects. Using targets to control the
linker at worst implies having one custom target per project.

Using different targets implies having different sysroots, one per target. This
is wasteful because custom targets are usually based on built-in targets and
just add linker arguments thus the generated sysroots will be exactly the same
because linker arguments don't affect code generation.

Using custom targets also makes Xargo a requirement as the a sysroot will have
to be compiled on the fly for each target. OTOH, if project local configuration
files are used *with* built-in targets then it'd be possible to *not* depend on
Xargo *if* the `rust-std` component is available for the built-in target.

Let me rephrase that last part:

If we settle on `.cargo/config` as the way to pass arguments to the linker then
most projects will switch back to built-in targets. If there's a `rust-std`
component available for those built-in targets then Xargo isn't required to
build the application and regular Cargo can be used.

# Blockers

For adopting this convention:

As `.cargo/config` passes arguments to the linker using the `rustc` interface,
currently it's only possible to *append* arguments to the linker invocation
(using `-C link-arg{,s}`). However, some applications require control over the
*order* of linker arguments; specifically, some arguments (object files) need to
be passed to the linker *before* the Rust object files. `rustc` doesn't support
this scenario ATM but a PR has been [sent] to allow it via the `-Z
pre-link-arg{,s}` arguments.

[sent]: https://github.com/rust-lang/rust/pull/41971

# Unresolved questions

- Does `.cargo/config` cover all the existing use cases?
  - Here's a [project] with a complex requirement of linker arguments and it's
    fully supported by the `.cargo/config` approach.

[project]: https://github.com/japaric/photon-quickstart/pull/6
