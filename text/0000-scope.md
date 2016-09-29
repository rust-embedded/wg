# The `embedded-rust` repository

The work of the embedded Rust initiative centers around this repository. This document describes its
scope and contents.

## Scope

This repository aims to be:

- The go-to source of information about Rust on embedded systems. You'll find documentation (links
  to guides, tutorials, etc.), a FAQ and a list of curated crates and "flagship" projects here. You
  can think of it as a [arewe$THINGyet] site minus the nice webpage.

- A place to collect and discuss "obstacles" that affect several projects, with the goal of:
  - Brainstorming and implementing solutions to problems that can be solved by the community, and
  - Informing the Rust team about high-priority issues that need to be addressed in the language,
    `rustc` or Cargo.

[arewe$THINGyet]: http://www.arewegameyet.org/

## Sub-ecosystems

The embedded ecosystem is vast; developing software for device A can be very different from
developing software for device B. To better categorize issues, crates and projects, we'll split the
ecosystem in "sub-ecosystems". Each sub-ecosystem represents a group of devices that: has similar
characteristics, shares a similar development process and can reuse software resources like crates
and tools.

Three sub-ecosystems will be used throughout this repository:

- The "Raspberry Pi" sub-ecosystem. The devices in this sub-ecosystem, usually:
  - have multi-core processors with core frequencies in the GHz range
  - have GBs of RAM
  - have GBs of persistent storage
  - run a "standard" OS (e.g. Debian)

- The "OpenWRT" sub-ecosystem. The devices in this sub-ecosystem, usually:
  - have a single-core processor whose frequency is in the sub-GHz range
  - have MBs of RAM
  - have MBs of persistent storage
  - run a "stripped down" version of a standard OS (e.g. OpenWRT)

- And, the "Arduino" sub-ecosystem. The devices in this sub-ecosystem, usually:
  - have a single-core processor whose frequency in the range of dozens of MHz
  - have KBs of RAM
  - have KBs of persistent storage
  - run powered by batteries (as in programs need to take energy consumption into account)
  - either operate without an OS (run "bare metal") or run a specialized, minimal OS, which can be
    of the "Real Time" variety (a RTOS)
  
## Contents

This repository will consist of: 

- A README with documentation, a FAQ and a list of curated crates and flagship projects for each
  sub-ecosystem.
- An issue tracker to discuss cross-project "obstacles".
- A Gitter channel where anyone can ask questions about Rust on embedded systems, and obstacles can
  be discussed more synchronously.
- And, a group of GitHub collaborators that:
  - manages the contents of the README
  - triages `upstream` issues, reports them to the Rust team and tracks their progress, and also
  - makes decisions about `meta` issues
  
Additional details about these items below:

### The README

The README will start with a description of the sub-ecosystems we introduced above. For each
sub-ecosystem, the README will also include:

- Documentation: Links to guides, tutorials and blog posts about using Rust on embedded systems.
- A FAQ: Questions that are asked repeatedly in our Gitter channel will be added to this FAQ.
- Curated crates: Crates that are commonly used go here. Specialized crates that are well documented
  and actively maintained also belong here.
- Flagship projects: These are projects that we want everyone to know about. Projects that are
  actively developed, contributor friendly and/or are a great example of Rust features in this
  space go in this list.

The exact mechanism by which the contents of the README will be selected is still **TBD** (*to be
determined*).

### The Issue tracker

We'll use this repository issue tracker to keep track and discuss "obstacles" that affect several
projects. An obstacle is anything that hinders the adoption or use of Rust in the embedded space.
Examples of obstacles below:

- Lack of documentation on how to write, flash and debug a Rust program written for a
  microcontroller.
- `rustc` doesn't support AVR, PIC, MSP430, Xtensa, etc. microcontrollers.
- Lack of embedded ["frameworks"]. More precisely, what's preventing them from coming into
  existence?
- Executables that link to `std` are "too big" (binary size wise).

["frameworks"]: http://platformio.org/frameworks

To continue with our sub-ecosystem motif we'll use the following labels to categorize obstacles:

- `rpi`
- `openwrt`, and
- `arduino`

Obstacles will by further categorized by "who has to solve them" with the following labels:

- The `community` label. Examples of `community` obstacles:
  - Agree on foundational crates
  - "standardize" (as in let's all recommend) some tool or project template
  - settle on best practices.

- And the `upstream` label. Obstacles in this category need to be addressed in the language,
  `rustc`, Cargo or in any other official tool. An example of an `upstream` obstacle:
  - [arduino] Cargo doesn't work out of the box: no built-in target definitions, no binary
    releases of `core`, support for linker scripts is not ideal, etc.

For `community` issues, we'll brainstorm solutions and then choose which solution we'll implement
via consensus. The exact mechanism is still **TBD** (but we'd probably adopt the [the RFC process]).

[the RFC process]: https://github.com/rust-lang/rfcs

For `upstream` issues, we'll use GitHub reactions (number of :+1:s) as a voting system to prioritize
them. Triaged issues will be brought up to the Rust team on a **TBD** basis. After an issue
has been escalated, we'll continue to post updates about upstream progress on the issue. And, once
the upstream solution has reached the stable channel, we'll close the issue.

The issue tracker will also be used to keep track of issues about our process. The `meta` label will
be used to identify those issues.

### Gitter channel

Our Gitter channel will adhere to Rust's [Code of Conduct].

[Code of Conduct]: https://www.rust-lang.org/en-US/conduct.html

### The Collaborators

The group of collaborators will consist of at least a **TBD** number of representatives from each
sub-ecosystem.

How the representatives will be selected is still **TBD**.

# Unresolved questions

- All the "**TBD**" processes.

- Do x86 kernel/OS projects belong in the embedded ecosystem? If you think that "x86 is not
  embedded" then check out the [Intel Quark]. In conclusion: Where do we draw the line?
  
[Intel Quark]: http://www.intel.com/content/www/us/en/embedded/products/quark/overview.html

- Should we use an IRC channel on Mozilla's network instead of Gitter?
  - good: there's already lots of people on Mozilla's IRC network; it's easier for them to join
    another IRC channel than it's for them to join Gitter.
  - good: don't need a GitHub account to participate.
  - bad: Logging has to be requested whereas Gitter has logging out of the box
  
- Should we use the users/internals forum instead of the issue tracker to discuss obstacles?
  - good: more exposure of our issues to the general community
  - bad: our topics can easily get lost in the stream of user posts

- Should we use the RFC process to pick a solution to `community` issues?
  - Is the RFC process too heavy-weight (as in too formal, requires too much effort or takes too
    much time (note that reaching consensus through any means takes time))?
  - We'd need to create a team, which is not necessarily the same as the collaborators to this
    repository, that makes the final decisions about RFCs.
  - What are alternatives to the RFC process that are also based on reaching consensus?

- Should we split the collaborators in teams that handle different tasks? One team for resources
  (the README) and another for `upstream` issues.
  
- Likewise, should we split this repository in two: one for resources and one for issues.

# Bikeshedding

- Better terms for "sub-ecosystem", "obstacles" and "flagship"?

- Should we use different names for the sub-ecosystems?
  - The chosen names are very reminiscent of the devices we are referring to but can they lead to
    copyright/trademark/IP/whatever problems?
  - One alternative: `microcontroller`, `router` (?) and `sbc` (Single Board Computer)?

- `rpi` or `raspberrypi` or `raspberry-pi` for the Raspberry Pi sub-ecosystem label?
