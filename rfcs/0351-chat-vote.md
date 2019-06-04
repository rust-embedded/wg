# Summary
[summary]: #summary

In order to select a new chat platform, we would like to put this up to a community vote on GitHub.

# Motivation
[motivation]: #motivation

As people may have noticed **irc.mozilla.org** which is the main communication channel for the rust-embedded WG and community is coming to an end, cf. [here](http://exple.tive.org/blarg/2019/04/26/synchronous-text/) and also [here](https://blog.rust-lang.org/2019/04/26/Mozilla-IRC-Sunset-and-the-Rust-Channel.html). While this change is going to take effect only in a few months from now this is the right time to decide which action to take forward and chose for a proper replacement.

This RFC would propose a method for selecting the new chat platform through a community vote. The outcome of this vote would be used for community and WG chat moving forward.

# Detailed design
[design]: #detailed-design

## Selection proposal

Some team members spent some effort identifying the most promising contenders which I propose should serve the base for a decision, unless additional suitable proposals are made (in alphabetical order):
* Discord
* Gitter
* IRC
* Matrix
* Zulip

## Community vote

As said it seems to make sense to sense to let the whole user base cast a vote on the preferred communication channel. The method this would work is by putting each candidate in its own post in a GitHub issue and, announce the vote on all available channels and count the number of all upvotes (👍) on each candidate after a fixed period of 2 weeks. Multiple choices are allowed and if the 2 highest results are within 3 votes of each other there will be a second ballot with only the 2 highest ranking options. In case of a real tie in that second ballot the @rust-embedded/core members will break the tie.

## Time to vote

The time to vote will end 2 weeks after RFC approval on Sunday.

# How We Teach This
[how-we-teach-this]: #how-we-teach-this

Since the agreed up new communication channel will affect how we communicate both as a working group but also with users we will need to change all mentions of the current channel on irc.mozilla.org, including but not limited to, our homepage at https://github.com/rust-embedded/wg and the #rust-embedded channel on IRC itself.

Also to reach the largest amount of interested folks we should announce change on our blog, forums and social media accounts.

# Drawbacks
[drawbacks]: #drawbacks

None known, giving power to the people is rarely a bad idea. To limit backlash and promotion of unsuitable channels the choices will be limited to sensible options.

# Alternatives
[alternatives]: #alternatives

* https://github.com/rust-embedded/blog/pull/51
    * Default move to Freenode RFC on a specific future date, requires an RFC to change

* Wait for Mozilla to make a decision and adopt it

# Unresolved questions
[unresolved]: #unresolved-questions

* What info is required for each choice?
* When to start vote?
