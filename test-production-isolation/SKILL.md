---
name: test-production-isolation
description: "Stop tests writing into real data. Withhold production credentials by location so a misdirected test cannot reach production even if it tries, rather than asking test code to behave. Use whenever a test performs a write, whenever you point a suite at an environment, whenever you add a 'safe mode' flag, and whenever real records appear that nobody created."
---

# Making a test physically unable to touch production

There was a record in the live database that did not exist.

It looked like all the others: right shape, plausible dates, sitting in the active list, being counted.
It had been created by a test proving that the "add a record" form worked — and the test had proved it
by adding one, to the real database, and then not cleaning up.

Its siblings numbered in the thousands. Most were soft-deleted and invisible on every screen, so
nothing ever looked wrong. Every total on every page had been slightly false for months, and no error
was ever raised, because from the software's point of view nothing had gone wrong at all.

## The rule

> **A test process must not hold credentials that reach production. Not "must not use them" —
> must not have them.**

Every alternative is a request for good behaviour from the exact code most likely to be wrong:

| approach | why it fails |
|---|---|
| a `--dry-run` flag | one code path forgets to check it |
| an `IS_TEST` environment variable | set in the shell that ran it, absent from the one that scheduled it |
| a naming convention for test records | works until cleanup fails, and cleanup fails |
| "the test points at staging" | a default, a fallback, or a redirect quietly resolves elsewhere |
| review and discipline | fine until the afternoon someone is in a hurry |

Each is a rule the test code must remember. The credential approach is a fact about the environment
the test runs in, and facts do not have a bad day.

## How to implement it

**1. Make the credential store location-aware.** Production secrets are granted only to processes
whose declared purpose is production work. Everything else — the test suite, exploratory scripts, the
agent's scratch runs — resolves to a disposable environment's credentials or to nothing at all.

The mechanism can be simple: a manifest listing every writer and the bucket it is allowed to reach,
consulted by whatever hands out credentials. What matters is that the list is **explicit, enumerable,
and fails closed**. A writer that is not on the list gets nothing. Silence must never mean permission.

**2. Give the disposable environment a real, complete copy of the schema.** The commonest reason
people point tests at production is that no other environment is realistic enough to be worth testing
against. That is a legitimate complaint, and it has to be answered with infrastructure, not with a
policy telling people to be careful.

**3. Prove the credentials are actually absent.** Add a check that runs *in the test environment* and
asserts that the production credential cannot be obtained. That check is the guard's own guard, and
without it you have a policy rather than a mechanism.

## Verify the effect, not the call

This is where isolation work usually stops one step too early.

"The redirect ran" and "the write went to the copy" are different claims. The first is about your
code; the second is about the world. Only the second is what you care about, and you can have the
first while the second is false — a fallback path, a cached connection, a client that reconnects with
different settings on retry.

After wiring up isolation, **read the production data and confirm nothing arrived.** Then break the
isolation deliberately and confirm something does. A guard tested in only one direction has not been
tested; you have confirmed it is capable of saying no, not that it says no on the right occasions.

## Keep one dumb ground-truth check

Alongside the suite, keep a small check that knows **nothing** about your tests and only compares real
data to itself: total record count, count of records with no creator, count created in the last day.
Run it on a schedule and alert on movement nobody explained.

Its whole value is that it shares no assumptions with the code it is watching. A sophisticated check
built on the same model of the system inherits the same blind spot — that is precisely how the
original contamination stayed invisible: every instrument that could have seen it was built by the
same hand, on the same beliefs.

## Cleaning up what is already there

Almost certainly there is existing contamination. Two rules:

- **Never delete by inference.** Do not build a heuristic for "looks like test data" and run it
  against real records. The false positive destroys something irreplaceable, and you will not find out
  for months.
- **Identify by provenance, not by shape.** Creation timestamp, creating process, a marker the test
  harness wrote. If a record cannot be positively attributed to a test, it stays, and the residue is
  documented rather than guessed at.

Where records are ambiguous, the honest outcome is a written note saying so. A count you can defend
beats a clean-looking number you cannot.

## Run the new guard inside the real pipeline before trusting it

A self-test proves the guard fires. It says nothing about what else it broke.

A credential gate is, by construction, in the path of everything. Run the full suite with it active
and read the failures individually: some will be the guard working, and some will be legitimate work
it has just blocked. Sorting those two apart is the actual job, and skipping it is how a correct guard
gets reverted within a week for being "broken".
