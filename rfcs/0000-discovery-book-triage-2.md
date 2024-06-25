- Feature Name: Embedded Discovery Book Triage
- Start Date: 2024-05-21
- RFC PR: #759
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Triage the current Embedded Discovery Books; a single BBC micro:bit v2
Embedded Discovery Book becomes the current active working version.

# Motivation
[motivation]: #motivation

Right now there are two "official" versions of the Embedded
Discovery Book (EDB) in a single repo. The older and
slightly more complete version is the original STM32
Embedded Discovery Book
([EDB-STM](https://docs.rust-embedded.org/discovery/f3discovery/))
for the [STM 32F303VC Discovery
Board](https://www.st.com/en/evaluation-tools/stm32f3discovery.html). The
newer version is the BBC micro:bit Embedded Discovery Book
([EDB-MB](https://docs.rust-embedded.org/discovery/microbit/)),
which covers both the micro:bit v1.5
([MB1](https://tech.microbit.org/hardware/1-5-revision/))
and the micro:bit v2.2X
([MB2](https://tech.microbit.org/hardware/)). (There are
also v1.00 and v2.00 Microbit boards, but the differences in
minor revisions are pretty negligible; EDB-MB ignores them.)

EDB-STM is not well-maintained. The underlying hardware was
not easily available during the pandemic. Further, there are
[reports](https://github.com/rubberduck203/stm32f3-discovery/issues/42)
that the IMU used in current versions of the STM 32F303VC
Discovery Board has been upgraded, meaning work would be
needed to distinguish the old and new boards and update
EDB-STM for the new board. What would happen to the docs for
the old board is questionable.

Having both MB1 and MB2 in EDB-MB is confusing. Futher, MB1
is no longer readily available, and there appear to be very
few of them in the wild. There are no known advantages to
working with an MB1 over an MB2, and both boards are very
cheap.

Finally, the EDB-MB also needs both some maintenance and an
ongoing maintenance plan and infrastructure.

Dealing with all of this is best suited to a multi-phase
project. This RFC proposes a "triage" phase in which things
are brought to a good current state. A later "development"
RFC will cover EDB improvements and an ongoing maintenance
plan.

# Detailed design
[design]: #detailed-design

1. Deprecate EDB-STM.

   * Close as many open EDB-STM issues as possible with
     either fixes or WONTFIX tags and an explanation.

   * Close all open EDB-STM PRs by either accepting or
     rejecting them with an explanation.

2. Do a minimum-effort split of the EDB repo into two book
   repos: a legacy repo and a new MB2-only repo.

   * Fork the EDB repo to create a separate EDB-MB2 repo. In
     this repo, remove the STM book and examples; remove all
     MB1 material and patch all example code to be
     MB2-specific.
 
   * Clearly mark the legacy repo as deprecated in the
     README; clearly mark EDB-STM and EDB-MB as deprecated
     on the first page; post an open issue explaining
     the deprecation.
     
   * Provide clear instructions on where to go for the
     current EDB-MB2 book.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Once this work is substantially completed (at least phases 1
and 2), we should publicize what we are doing widely. This
will increase the chances of a good initial developer
experience.

# Drawbacks
[drawbacks]: #drawbacks

* There will be some confusion about which book to use for
  MB2. Adequate signposting will be necessary.

* Issues and PRs for MB2 will continue to be posted on the
  legacy repo. This will necessitate redirecting people.

# Alternatives
[alternatives]: #alternatives

* Continue to maintain a Discovery Book for all three
  currently-documented boards. This could be done by
  continuing as-is, with a MB1/MB2 split, or with a merge to
  create a single EDB. A single EDB might be harder to
  navigate than a board-specific one.
  
  This is a lot of effort to do and maintain: it would
  probably require sustained work by a large team.

* Do some fancier repo split. This is what was originally
  proposed, but drawbacks with this approach emerged,
  including potential loss of legacy material and breaking
  existing documewtation links.

* Do a new EDB or other beginner book "from scratch". This
  is a great long-term goal, but leaves things in a
  less-than-perfect state for the time being. It is also
  huge work.

# Unresolved questions
[unresolved]: #unresolved-questions

The proposed upcoming "development" RFC will address larger
issues with the EDB. This "triage" RFC is intended to put
things in a good place for that discussion.
