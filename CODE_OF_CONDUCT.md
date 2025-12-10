# The Rust Code of Conduct

## Conduct

**Contact**: [Embedded WG][wg]

* We are committed to providing a friendly, safe and welcoming environment for all, regardless of level of experience, gender identity and expression, sexual orientation, disability, personal appearance, body size, race, ethnicity, age, religion, nationality, or other similar characteristic.
* On Matrix and IRC, please avoid using overtly sexual nicknames or other nicknames that might detract from a friendly, safe and welcoming environment for all.
* Please be kind and courteous. There's no need to be mean or rude.
* Respect that people have differences of opinion and that every design or implementation choice carries a trade-off and numerous costs. There is seldom a right answer.
* Please keep unstructured critique to a minimum. If you have solid ideas you want to experiment with, make a fork and see how it works.
* We will exclude you from interaction if you insult, demean or harass anyone. That is not welcome behavior. We interpret the term "harassment" as including the definition in the [Citizen Code of Conduct](http://citizencodeofconduct.org/); if you have any lack of clarity about what might be included in that concept, please read their definition. In particular, we don't tolerate behavior that excludes people in socially marginalized groups.
* Private harassment is also unacceptable. No matter who you are, if you feel you have been or are being harassed or made uncomfortable by a community member, please contact one of the channel ops or any of the [Embedded WG][wg] immediately. Whether you're a regular contributor or a newcomer, we care about making this community a safe place for you and we've got your back.
* Likewise any spamming, trolling, flaming, baiting or other attention-stealing behavior is not welcome.

## Moderation

These are the policies for upholding our community's standards of conduct.

1. Remarks that violate the Rust standards of conduct, including hateful, hurtful, oppressive, or exclusionary remarks, are not allowed. (Cursing is allowed, but never targeting another user, and never in a hateful manner.)
2. Remarks that moderators find inappropriate, whether listed in the code of conduct or not, are also not allowed.
3. Moderators will first respond to such remarks with a warning.
4. If the warning is unheeded, the user will be "kicked," i.e., kicked out of the communication channel to cool off.
5. If the user comes back and continues to make trouble, they will be banned, i.e., indefinitely excluded.
6. Moderators may choose at their discretion to un-ban the user if it was a first offense and they offer the offended party a genuine apology.
7. If a moderator bans someone and you think it was unjustified, please take it up with that moderator, or with a different moderator, **in private**. Complaints about bans in-channel are not allowed.
8. Moderators are held to a higher standard than other community members. If a moderator creates an inappropriate situation, they should expect less leeway than others.

In the Rust community we strive to go the extra step to look out for each other. Don't just aim to be technically unimpeachable, try to be your best self. In particular, avoid flirting with offensive or sensitive issues, particularly if they're off-topic; this all too often leads to unnecessary fights, hurt feelings, and damaged trust; worse, it can drive people away from the community entirely.

And if someone takes issue with something you said or did, resist the urge to be defensive. Just stop doing what it was they complained about and apologize. Even if you feel you were misinterpreted or unfairly accused, chances are good there was something you could've communicated better — remember that it's your responsibility to make your fellow Rustaceans comfortable. Everyone wants to get along and we are all here first and foremost because we want to talk about cool technology. You will find that people will be eager to assume good intent and forgive as long as you earn their trust.

The enforcement policies listed above apply to all official embedded WG venues; including the official Matrix room (#rust-embedded:matrix.org) and linked IRC channels (#rust-embedded on Libera); and all GitHub repositories under rust-embedded.

## AI Tool Use Policy

When using AI tools for contributing, human oversight remains critical.

- **You are responsible for your contributions.** It is your responsibility to review, test, and understand everything you submit. Submitting unverified or low-quality machine-generated content creates an unfair review burden on the community and is not an acceptable contribution. Contributors should review and understand their own submissions before asking the community to review their code.
- **Start with small contributions.** Open source communities operate on trust and reputation. Reviewing large contributions is expensive, and AI tools tend to generate large contributions. We encourage new contributors to keep their first contributions small, specifically below 150 additional lines of non-test code insertions, until they build personal expertise and maintainer trust before taking on larger changes.
- **Be transparent about your use of AI.** When a contribution has been significantly generated by an AI tool, we ask you to note this in your pull request description. This transparency helps the community develop best practices and understand the role of these new tools.
- **Limit AI tools for reviewing.** As with creating code, reviewers may use AI tools to assist in providing feedback, but not to wholly automate the review process. Particularly, AI may not make the final determination on whether a contribution is accepted. Reviewers must disclose the use of AI tools and avoid using them if asked to do so by the contributor.

This policy extends beyond code contributions and includes, but is not limited to, the following kinds of contributions:

- Code, usually in the form of a pull request.
- RFCs or design proposals.
- Issues or security vulnerabilities.
- Comments and feedback on pull requests.

### Extractive Changes

Sending patches, PRs, RFCs, comments, etc., is not free – it takes a lot of maintainer time and energy to review those contributions! We see the act of sending low-quality, un-self-reviewed contributions to the Rust Embedded WG project as “extractive.” It is an attempt to extract work from the community in the form of review comments and mentorship, without the contributor putting in comensurate effort to make their contribution worth reviewing.

Our golden rule is that a contribution should be worth more to the project than the time it takes to review it. If a maintainer judges that a contribution is extractive (i.e., it is generated mainly with tool-assistance or requires significant revision), they
should copy-paste the following response, add the `extractive` label if applicable, and refrain from further engagement:

> This PR appears to be extractive, and requires additional justification for why it is valuable enough to the project for us to review it. Please see our developer policy on AI-generated contributions: https://github.com/rust-embedded/wg/blob/master/CODE_OF_CONDUCT.md#AI-Tool-Use-Policy

Other reviewers should use the label to prioritize their review time.

The best ways to make a change less extractive and more valuable are to reduce its size or complexity or to increase its usefulness to the community. These factors are impossible to weigh objectively, and our project policy leaves this determination up to the maintainers of the project, i.e. those who are doing the work of sustaining the project.

If a contributor responds but doesn’t make their change meaningfully less extractive, maintainers may lock the conversation.

### Copyright

AI systems raise many questions around copyright that have yet to be answered. Our policy on AI tools is similar to our copyright policy: Contributors are responsible for ensuring that they have the right to contribute code under the terms of our license, typically meaning that either they, their employer, or their collaborators hold the copyright. Using AI tools to regenerate copyrighted material does not remove the copyright, and contributors are responsible for ensuring that such material does not appear in their contributions. Contributions found to violate this policy will be removed just like any other offending contribution.

*Adapted from the [Node.js Policy on Trolling](http://blog.izs.me/post/30036893703/policy-on-trolling) as well as the [Contributor Covenant v1.3.0](https://www.contributor-covenant.org/version/1/3/0/).*

*AI Tool Use Policy adapted from the proposed [LLVM AI tool policy: start small, no slop](https://discourse.llvm.org/t/rfc-llvm-ai-tool-policy-start-small-no-slop/88476)*

[wg]: https://github.com/rust-embedded/wg#organization
