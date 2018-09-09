# Summary
[summary]: #summary

This RFC proposes updating our voting guidelines to make it easier for
uncontroversial proposals to be accepted as the number of members in
the working group grows. Specifically, the number of approving votes
required will remain 50% of the members for one week, then be decreased
to 33% for two weeks, and then if no concerns are raised any number of
approvals will accept the proposal.

Considerations are also made for notifying working group members of
outstanding voting issues.

# Motivation
[motivation]: #motivation

Currently we require a [voting majority](https://github.com/rust-embedded/wg/blob/master/rfcs/0136-teams.md#voting-majority)
consisting of more approvals than abstensions and zero unresolved concerns.
In other words, we currently require approval from more than 50% of the
current membership of the concerned team.

At present several issues have been difficult to pass, not because of any
concerns raised but because our membership has grown and therefore so too has
the number of required votes. See for example issues #172, #187, #190.

Several members have commented that they do not want their lack of voting
to block an issue, but they are just not interested in every outstanding
issue or do not have time to give them all detailed consideration.

We would like to be able to pass sub-group and working-group votes quickly
where they are reasonably uncontroversial, without having to chase up many
members to give a perfunctory vote.

# Detailed design
[design]: #detailed-design

## Majority

For the first week after an issue is opened, the current 50% threshold remains
in place. Votes which receive sufficient approvals can be accepted as usual.

After one week has passed, the threshold is reduced to 33%, and remains at 33%
for two weeks. There must still be zero unresolved concerns; but only 33% of
the members need to approve for a proposal to be accepted.

After those two weeks have passed (three weeks in total), _any_ number of
approvals will permit a vote to be accepted, so long as there are no
unresolved concerns. A member is explicitly permitted to raise a concern
that there are not sufficient votes or an issue has not received sufficient
attention and thereby block it from being accepted until that concern is
addressed.

## Notifying Members

At each weekly meeting, outstanding votes will be identified, and members
present at the meeting reminded to consider voting. After each meeting,
all members of the relevant group will be emailed with a reminder of any
pending votes.

By notifying members of outstanding votes each week, it is hoped that every
member will have sufficient time to consider whether they have concerns over
a specific vote. If the full three weeks elapses with no concerns raised,
we conclude no members have concerns.

# Alternatives

## Status Quo

We could maintain the current system, perhaps just adding the reminder emails
to attempt to collect sufficient voting majorities.

## Different Thresholds

The 33% number could be changed to 25%, or could be 33% for one week and then
25% for the second week.

## Longer Timespans

We could increase the two-week 33% period to three weeks or longer.

## No Small Approvals

We could omit the final stage where any number of approvals accepts a vote,
always requiring at least the 33% (or 25%) majority.
