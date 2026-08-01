# Agent skills

Eight small, self-contained skills for AI coding agents. Each is a single markdown file: when to invoke
it, what to do, and why the rule exists.

They were extracted from one long-running project where an agent does most of the work and a
non-programmer owns the outcome. Every one of them exists because something went wrong first.

| skill | what it is for | the story behind it |
|---|---|---|
| [`citations`](citations/SKILL.md) | Never cite a source you have not fetched and read. Fabricated identifiers are well-formed, and the worst case is a *real* paper that does not say what it is cited for. | [The reference that didn't exist](https://blonderoofrat.com/the-reference-that-didnt-exist/) |
| [`denominator-rule`](denominator-rule/SKILL.md) | Never trust a green light that cannot state its denominator. A check that found nothing and a check that checked nothing print the same thing. | ["Zero problems found." Zero of what?](https://blonderoofrat.com/zero-problems-found-zero-of-what/) |
| [`hook-tuning`](hook-tuning/SKILL.md) | Block only on deterministic facts; warn and log on judgment. A false positive in a blocker is a defect, because it trains the bypass reflex. | [The safety check we learned to ignore](https://blonderoofrat.com/the-safety-check-we-learned-to-ignore/) |
| [`model-provenance`](model-provenance/SKILL.md) | A model's self-reported identity is not evidence. Verify which model produced an output from metadata, never from the text. | |
| [`wp-inline-js-safety`](wp-inline-js-safety/SKILL.md) | WordPress silently kills an inline script when `&&` sits between a `<` and a `>`. Invisible server-side. | [The page looked perfect. One button just did nothing.](https://blonderoofrat.com/the-page-that-looked-perfect-and-did-nothing/) |
| [`wp-admin-script-leak`](wp-admin-script-leak/SKILL.md) | A front-end JavaScript error visible only when logged in. A logged-out check cannot see it. | |
| [`windows-silent-subprocess`](windows-silent-subprocess/SKILL.md) | Stop spawned processes popping console windows on Windows and stealing the user's keystrokes. | [The window that typed my words into the wrong place](https://blonderoofrat.com/the-window-that-typed-my-words-into-the-wrong-place/) |
| [`agent-clean-room`](agent-clean-room/SKILL.md) | Your agent sends your branch, your diff and your project files alongside your question. A second model can decline the lot because of what your repository is about. Run it from a scratch directory instead. | |

Where a skill has a story, it is the long-form account of the incident that produced the rule —
what actually broke, what it looked like at the time, and why the rule is shaped the way it is.
The skill file is the rule; the article is the scar.

## The thread running through all of them

Most of these are about the same failure in different clothes: **something reports success while not
working.** A guard that scanned zero files. A citation that resolves to the wrong paper. A review
attributed to a model that never ran. A script that was silently corrupted after your code emitted it.

Ordinary bugs announce themselves. These do not: they produce a green light, which is the one signal
nobody investigates. That is why each of these is a habit rather than a tool.

## Status and support

**Provided as-is. Not maintained.** These are frozen snapshots of rules that work in one project.
There is no roadmap, no versioning promise, and issues are not monitored.

**Fork them.** They are short on purpose so that adapting one is easier than filing a request about
it. If a rule is wrong for your situation, change it: several are deliberately opinionated in ways
that will not suit everyone.

Each file carries the date it was last revised. If that date is old, assume the surrounding tooling
has moved and check the specifics before relying on them; the reasoning ages better than the endpoints.

## Licence

CC0 1.0 (public domain dedication). See [`LICENSE`](LICENSE). Copy, adapt and redistribute without
attribution or permission.
