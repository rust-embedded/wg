# Summary
[summary]: #summary

This RFC proposes establishing a new repository for a web page advertising
noteworthy embedded Rust projects. This new resource would go beyond the
existing [awesome-embedded-rust] list by allowing long form
descriptions, photographs, and videos to best demonstrate the projects.

[awesome-embedded-rust]: https://github.com/rust-embedded/awesome-embedded-rust

# Motivation
[motivation]: #motivation

We would like a well-presented list of interesting embedded projects using Rust
to advertise Rust's abilities in this domain. Existing lists are mainly
targeted at embedded Rust developers looking for libraries; this list would
instead be advertising the final projects people have accomplished with Rust,
and so should appeal to potential users who are not already embedded Rust
developers.

The main embedded Rust website will contain a shortlist of especially notable
projects, but is aimed at presenting a very high-level overview to all users
including non-technical users. This proposed list would be able to present more
projects in more detail, and will require that their source code is publically
available so embedded developers interested in Rust can inspect it.

# Detailed design
[design]: #detailed-design

## New Repository

We create a new repository, `embedded-rust-showcase` (TBC), in the
rust-embedded organisation and under the existing Resources team. It will use
the same underlying technology as the upcoming embedded Rust website to render
Markdown to a web page.

## New Web Page

The new web page would live at https://showcase.embedded-rust.org (TBC).

Each project is listed with a photograph/GIF/video, description, and other
details (see Submission section).

Additionally, each project has a badge indicating whether it builds on stable
Rust or requires a nightly compiler.

## Project Requirements

To be considered for inclusion, projects must:

* Involve embedded Rust, in other words, use Rust and run on embedded hardware

* Have publically available source code

The objective of this list is to showcase Rust code in action; we can't do this
if people can't read the code! There might be space on the main Embedded Rust
website to showcase projects known to use Rust but without public code. We
don't require that the source code is under an open source license; just that
it's available for interested users to read.

* Have working CI builds

Since we want people to be able to learn from the code, we require it at least
builds successfully. Working CI also shows what versions of Rust it builds on,
which is useful to establish if a project works on stable Rust.

* Have at least one photograph/video/GIF of the project in action

## Project Submission

Projects are submitted by their authors via pull requests. Submissions must
contain the following information:

* Project name
* Author name
* Project website/repository
* Project description
* Images, GIFs, or video of the project
* Whether or not the project builds on stable Rust

After a project is submitted, the resources team will review the PR and
vote on inclusion. They may consult other relevant teams in the process,
but final approval of submissions is at the discretion of the resources team.

# Alternatives

* Instead of badges for builds-on-stable projects, the nightly-only projects
  could be binned together at the bottom of the page.
* The web page could instead be maintained as part of the main embedded-rust
  website, existing as a page in that website.
* We could extend awesome-embedded-rust to permit longer descriptions of
  projects.
* We could integrate this concept into the existing awesome-embedded-rust
  repository, but still render the projects to a web page.

# Unresolved Questions
[unresolved]: #unresolved

* Final name for repository/web page
* The specific technical solution for rendering the repository contents to a
  web page
