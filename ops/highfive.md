# (Using) Highfive

Reference: https://github.com/rust-lang-nursery/highfive

- When you open a PR you will get a welcome comment from `@rust-highfive`
  assigning a reviewer to your PR.

- The reviewer will be picked at random from the team assigned to the
  repository.

- You can override the reviewer using `r? @someone` in any comment. If `r?` is
  used in the PR description you will *not* get a comment from `@rust-highfive`,
  but `@someone` will still be assigned to the PR.

- If you use `r? @ghost` in the PR description no one will get assigned to
  review the PR.

- If you run into what appears to be a bug in highfive please notify the
  rust-lang/infra team (#infra channel on discord) ASAP -- the highfive logs are
  kind of short lived.
  
# Maintaining Highfive

The list of reviewers is not taken from the team member list in GitHub but needs
manual maintenance per PR to this configuration upon membership change:

https://github.com/rust-lang/highfive/blob/master/highfive/configs/_global.json
