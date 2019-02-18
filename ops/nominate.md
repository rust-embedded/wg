# Nominating rust-lang/* issues

This document describes the process of "nominating" Cargo, compiler and `std`
library issues to their respective Rust teams. Nominating here means adding the
issue to the list of issues the team triages on a (bi)weekly basis so it can be
assessed and prioritized.

## Cargo

In the case of bugs in Cargo:

- On a new or existing issue in rust-lang/cargo leave a comment describing how
  this bug affects embedded developers and mentioning that you want to nominate
  the issue; add the `I-nominated` label to the issue (omit this step if you
  don't have the permissions to do so); and cc one of the Cargo team members.

The Cargo team meets on Wednesdays 15:00 (US) Central Time in the #cargo
(Discord) channel.

## Compiler

In the case of bugs in the compiler:

- On a new or existing issue in rust-lang/rust leave a comment describing how
  this bug affects embedded developers and mentioning that you want to nominate
  the issue; add the `I-nominated` and `T-compiler` labels to the issue (omit
  this step if you don't have the permissions to do so); and cc one of the
  compiler team members.

- After the issue is triaged by the compiler team it will be assigned a
  priority. `P-high` issues block the next stable release; `P-medium` issues
  don't. In either case a person will be assigned to the issue if it's
  actionable.

- If no progress is seen after, say, 2 weeks it's OK to ping the person assigned
  to the issue.

The compiler team meets on Thursdays 16:00 Central European Time on Zulip.

## Libs

In the case of an API that we want to see stabilized:

- If it's ready and just needs a decision, on the tracking issue in
  rust-lang/rust leave a comment asking to start the stabilization (FCP)
  process; add the `I-nominated` and `T-libs` labels to the tracking issue (omit
  this step if you don't have the permissions) and cc a member of the libs team.

- If it needs implementation work then that needs to happen before pinging the
  libs team. "PRs welcome".

- If there are design decisions without consensus then there's no solution.
  "Wrangling" consensus is the best option. Re-pinging the issue is acceptable
  if there has been no progress for a while.

The libs team triages issues every two weeks.
