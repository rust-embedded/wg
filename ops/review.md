# Reviewer handbook

These apply to both team members and collaborators.

We use https://bors.tech in all our repositories. Bors commands are documented in
https://bors.tech/documentation/

All PRs must be approved using [GitHub reviews] before you can send the PR to bors.

[GitHub reviews]: https://help.github.com/articles/approving-a-pull-request-with-required-reviews/

Most PRs can be approved by a single team member or collaborator, but PRs labeled `needs-decision`
need to be approved by the whole team (see [voting majority]) before they are sent to bors.

- Apply the `needs-decision` label to all PRs that include breaking changes and / or major changes
  that have not been previously discussed and approved.

- bors will ignore `r+` commands on PRs that have the `needs-decision` label.

- Remove the `needs-decision` label after voting majority has been achieved.

- At this point you can review the PR and send it to bors.

[voting majority]: https://github.com/rust-embedded/wg/blob/master/rfcs/0136-teams.md#voting-majority
