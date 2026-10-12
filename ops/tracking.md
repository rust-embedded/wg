# Tracking progress of yearly goals

At the beginning of the year the WG decides on the goals that it plans to
achieve by the end of the year. This document describes how the progress of
those goals is tracked in the rust-embedded/* issue trackers.

- One tracking issue per goal. All these issues will be under the `$YEAR`
  milestone.

- All tracking issues have at least one team associated to it (\*). This will be
  reflected using a label, e.g. `T-cortex-m`. (\*) With the exception of the
  meta issue used to track upstream (rust-lang/*) bug fixes, stabilization
  issues, etc.

- All issues will have at least one WG member assigned to it (using GH
  "assignees" feature).

- No member shall be assigned to more than three tracking issues at any point in
  time.

- The assignee(s) will report updates on the tracking issues as comments. In
  particular, they'll post a summary report each week, before the weekly
  meeting. No summary report is needed for "not started" issues.

- The summary reports will start with a status label indicating whether the goal
  is "on track", "blocked" or in the "help wanted" state. The summary will
  include the updates since the last summary report and the next steps for the
  issue (e.g. write an RFC, or wait for FCP to end). Example:

``` markdown
Status: on track

Updates: An RFC was submitted.

Next steps: wait for T-libs approval.
```

- If the status of the issue hasn't changed since last week the assignee can
  skip the summary report of that week but they must write a summary report the
  next week.

- Summary reports will be linked from the issue description (i.e. from the top
  of the thread) in reverse chronological order using the following format:
  `W$N: $STATUS`, where `$N` is the week number and `$STATUS` is the status
  label. Example:

``` markdown
Title: ARM intrinsics

### Status reports

- [W13: blocked](..)
- [W11: on track](..)
- W6-W10: not started

---

### Description

This tracking issue is about stabilizing ARM Cortex-M / Cortex-R / Cortex-A
intrinsics (e.g. `wfi`) in the `core::arch::arm` module.
```

- If an issue has been in "blocked" or "help wanted" state for more than two
  weeks then it will automatically be scheduled for discussion during the next
  weekly meeting. At least one member of the team assigned to the issue must
  attend that meeting.
