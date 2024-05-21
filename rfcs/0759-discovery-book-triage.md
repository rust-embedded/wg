- Feature Name: Embedded Discovery Book Triage
- Start Date: 2024-05-21
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary
[summary]: #summary

Triage the current Embedded Discovery Books; a single BBC micro:bit v2
Embedded Discovery Book becomes the current active working version.

# Motivation
[motivation]: #motivation

Right now there are two "official" versions of the Embedded
Discovery Book (EDB). The older and slightly more complete
version is the original STM32 Embedded Discovery Book
(EDB-STM) for the STM 32F303VC Discovery Board. The newer
version is the BBC micro:bit Embedded Discovery Book
(EDB-MB), which covers both the micro:bit v1 (MB1) and the
micro:bit v2 (MB2).

EDB-STM is not well-maintained. The underlying hardware was
not easily available during the pandemic. Further, there are
reports that the IMU used in current versions of the STM
32F303VC Discovery Board has been upgraded, meaning work
would be needed to distinguish the old and new boards and
update EDB-STM for the new board. What would happen
to the docs for the old board is questionable.

Having both MB1 and MB2 in EDB-MB is confusing. Futher, MB1
is no longer readily available, and there appear to be very
few of them in the wild.

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

   * Close all open issues with either fixes or WONTFIX tags
     and an explanation.

   * Close all open PRs by either accepting or rejecting
     them with an explanation.

   * Clearly mark the repository as deprecated in the
     README; clearly mark the Book as deprecated on the
     first page; post a single open issue explaining the
     deprecation. Provide clear instructions on where to go
     for current information.

2. Do a minimum-effort split of EDB-MB into two books: one
   for MB1 and one for MB2.

   * Fork the EDB-MB repo to create a separate MB1 repo. In
     this repo, remove all MB2 material and patch all
     example code to be MB1-specific.
 
   * Go through the same steps with the new EDB-MB1 repo
     that were gone through with STM. Note that there will
     be no initial open issues or PRs in the EDB-MB1 repo
     because of the fork: this is as desired.
     
   * The EDB-MB repo now becomes the EDB-MB2 repo. Remove
     all MB1 material and patch all example code to be
     MB2-specific. Try to close as many issues and PRs as
     are easily feasible.

3. Do as direct a port as possible of EDB-STM content
   "missing" from EDB-MB2. EDB-MB2 should cover all major
   topics covered by EDB-STM.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Once this work is substantially completed (at least phases 1
and 2), we should publicize what we are doing widely. This
will increase the chances of a good initial developer
experience.

# Drawbacks
[drawbacks]: #drawbacks

* People with STM32F3V303 and MB1 boards will feel
  disenfranchised by the deprecation.

* For a "triage" this is a reasonably high-effort
  approach. One can imagine some smaller cleanup that would
  be adequate.

# Alternatives
[alternatives]: #alternatives

* Continue to maintain a Discovery Book for all three
  currently-documented boards. This could be done by
  continuing as-is, with a MB1/MB2 split, or with a merge to
  create a single EDB. A single EDB might be harder to
  navigate than a board-specific one.
  
  This is a lot of effort to do and maintain: it would
  probably require sustained work by a large team.

* Do a new EDB or other beginner book "from scratch". This
  is a great long-term goal, but leaves things in a
  less-than-perfect state for the time being. It is also
  huge work.

# Unresolved questions
[unresolved]: #unresolved-questions

The proposed upcoming "development" RFC will address larger
issues with the EDB. This "triage" RFC is intended to put
things in a good place for that discussion.
