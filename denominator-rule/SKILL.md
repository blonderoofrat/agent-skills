---
name: denominator-rule
description: "Never trust a green light that cannot state its denominator. Use whenever you write, review or rely on a check, guard, lint, test sweep or audit; whenever a tool reports 'no problems found', 'all clear' or '0 issues'; and whenever a result seems too easy. A check that finds nothing and a check that checks nothing produce identical output."
---

# The denominator rule

**A check that found nothing and a check that checked nothing print the same thing.**

That sentence is the whole skill. Everything below is how to tell those two apart, and why the
distinction is worth building machinery around.

## The failure

You run a sweep. It says:

```
✓ No issues found.
```

Two worlds produce that output. In one, the tool examined every file there was and all of them were
fine. In the other, a path was wrong, a glob matched nothing, an API returned an empty list, a filter
excluded everything, an exception was swallowed, and the tool examined **zero** files and reported
success, because zero problems were found in zero things.

The second world is indistinguishable from the first *from the output alone*, and it is far more
common than people expect, because every one of its causes is an ordinary bug:

- a path or pattern that no longer matches anything, after a move or a rename;
- a filter that is subtly wrong, so everything is excluded;
- an API that returns `[]` on error instead of raising;
- a `try/except` that swallows the failure and continues;
- an auth failure that yields an empty list rather than an error.

None of these announce themselves. They all just look like good news. And a green light is the one
signal nobody investigates, so the failure persists exactly as long as nobody happens to notice.

## The rule

**Every check must report the size of the population it examined, and must fail when that population
is empty or implausible.**

```
✓ No issues found in 1,204 files.        ← trustworthy
✓ No issues found.                       ← says nothing
✗ REFUSED: matched 0 files. Expected hundreds. Not a pass.
```

(That expected floor is itself hand-kept, which rule 1 below warns against, but when it drifts it
fails *loudly*, which is the safe direction. A stale allowlist fails silently; a stale expectation
shouts. That asymmetry is the whole principle in miniature.)

Three requirements, in order of importance:

**1. Enumerate from the real boundary.** The denominator must come from the authoritative source, not
from a list maintained beside it. A hand-kept list of things-to-check drifts from reality silently,
and then the sweep is complete *with respect to the list* while missing what was added last month.
Enumerate from the filesystem, the schema, the registry, the API: the thing that cannot lie about
what exists.

**2. Report the count, always, including on success.** Not in verbose mode. Not in the logs. In the
one line a human reads. A number that only appears on failure cannot warn you that it is wrong.

**3. Fail closed on empty.** Zero examined is a refusal, never a pass. If zero is genuinely possible
and fine, say so explicitly *and say how you know*: "0 of 0 migrations pending, and the migrations
table exists" is a real result; a bare "0 issues" is not.

This one is not novel, and it is worth knowing that: `pytest` exits with code 5 when it collects zero
tests, for exactly this reason. The rule here is that the same reflex should apply to *every* green
light you produce or trust, not only to the ones a mature tool already handles for you.

## Prove it can fail

A check nobody has watched refuse is not a check. Before you trust a new guard:

**Break something on purpose and confirm the guard goes red.** Plant the exact defect it exists to
catch, run it, watch it fail, then remove the defect. Keep that fault-injection as a test if you can.

This is the single highest-value habit in this skill. Many guards have never once fired, and a guard
that has never fired is indistinguishable from a guard that cannot fire.

Watch especially for assertions that cannot fail:

```python
assert len(results) >= 0      # always true
assert True                   # ...yes
if items: check(items)        # silently skips when the list is empty
```

## Applying it to something you did not write

When a tool tells you everything is fine, ask three questions before believing it:

1. **How many things did it look at?** If it cannot say, its result is not evidence.
2. **Where did that list come from?** If from a hand-maintained file, when was that file last
   reconciled against reality?
3. **What would this look like if it were broken?** If the answer is "the same", you have learned
   nothing from the green light.

## The same rule in other clothes

- **Coverage:** "all tests pass" is meaningless without how many ran. A test file that fails to import
  is zero tests passing.
- **Search and sweeps:** "no matches" from a search whose path was wrong is not absence of the thing.
- **Permissions and audits:** "no unauthorised access found" across zero enumerated resources is the
  most dangerous form, because it reads as a security assurance.
- **Migrations and data:** "0 rows needed fixing" when the query targeted the wrong table.
- **Model and agent output:** "I checked all the files" is a claim about a denominator the model
  cannot see. Ask which files, and count them.

## What is and is not new here

The parts are all prior art, and you should know which before you repeat any of it as insight:
fail-closed-on-empty ships in `pytest`; reporting row counts is standard data-quality practice;
"prove it can fail" is the idea behind mutation testing; and the logic underneath is vacuous truth,
which is very old. What this skill adds is the *binding*: one rule, one word, applied to every green
light regardless of domain, so that it becomes a reflex rather than a thing you knew and still got
burned by.
