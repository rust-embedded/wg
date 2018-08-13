# IRC Operational Notes

The Rust Embedded Working Group uses the following IRC Channel for communication:
- Network: `irc.mozilla.org`
- Channel: `#rust-embedded`

This document concerns the organization of this channel. We adopted the hierarchy as follows in the rest of this document.

### Channel Owner

IRC user mode: `+q`

Rights:
 - Modify the list of admins, operators, half-operators, and voices.
 - Modify the topic of the channel.
 - Can mute the channel with `+m` channel mode.
 - Grant operator, half-operator and voice rights.
 - Invite bots into the channel.
 - Kick and ban users in violation of the channel rules or the Code of Conduct.
 - Speak when the channel is muted.

The channel owner for the Embedded WG's IRC channel shall be the leader of the Working Group (currently @japaric).

### Channel Operator

IRC user mode: `+o`

Rights:
 - Modify the list of half-operators and voices.
 - Modify the topic of the channel.
 - Can mute the channel with `+m` channel mode.
 - Grant half-operator and voice rights.
 - Invite bots into the channel.
 - Kick and ban users in violation of the channel rules or the Code of Conduct.
 - Speak when the channel is muted.

The channel operators should people who are maintaining projects under the umbrella of the `rust-embedded` organization.

### Channel Half Operator

IRC user mode: `+h`

Rights:
 - Grant voice rights to users.
 - Kick and ban users in violation of the channel rules or the Code of Conduct.
 - Modify the topic of the channel.
 - Speak when the channel is muted.

The channel's half-operators should be the members of the Triage team and esteemed contributors to the projects maintained under the `rust-embedded` organization. The decision as to who is an 'estemeed' contribution to a project is left to the discretion of each project maintainer.

### Channel Voice

IRC user mode: `+v`

Rights:
 - Speak when the channel is muted

The voice right is granted to volunteers and contributors of the WG at the discretion of project maintainers.
