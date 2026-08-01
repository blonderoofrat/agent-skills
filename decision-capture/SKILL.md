---
name: decision-capture
description: "Record a human's decisions so they are not silently re-litigated. Capture the decision in their own words, in one place, the moment it is made; gate re-asking on a lookup so the same question cannot be asked twice. Use whenever a human answers a question that constrains future work, whenever you are about to ask a question that feels familiar, and whenever you find two documents that disagree about what was decided."
---

# Capturing a decision so it stays decided

An answer that only went into the work has not been recorded. It has been *used*. Those are different
things, and the difference does not show up until weeks later, when the same situation comes round
again and there is nothing to look it up in.

At that point you ask again. And because you are asking rather than reading, you do not ask the same
question — you ask a slightly different one, shaped by whatever you are doing that day. You get a
slightly different answer, because it is a slightly different question. Now there are two answers,
both genuine, and no way to tell which one is current.

Nothing was overwritten. Nothing was lost. The decision simply never existed as a thing that could be
consulted, so it got made again, worse.

## The four rules

### 1. A decision is one record, in their words, in one place

Everything else points at it. Not a copy in the planning document and another in the code comment and
a third in a summary — those drift independently, and the drift is invisible because each of them
looks authoritative on its own.

Store the **verbatim words**, not your paraphrase. A paraphrase is a lossy summary made by the party
who will later be interpreting it, which is exactly the wrong party. It also loses hedges, and hedges
are load-bearing: "yes, but only while we are small" is a different decision from "yes."

Record alongside it: **when**, **where it came from** (a message, a call, a comment), and **what
question was actually asked**. A bare answer with no question attached is close to useless a month
later, because you cannot tell what it was ruling out.

### 2. Read the whole comment, never just the button

If your interface offers choices, the choice is the *least* informative part of the response.

People click an option and then qualify it — "this one, but not for anything already published", "yes,
though I would want to see it first." The qualification is frequently a different decision from the
button, sometimes the opposite of it. An automated reader that stores the selection and drops the
prose will be confidently wrong, and will stay wrong, because the record now says something the person
never meant.

Store the free text as the answer. Treat the button as an index, not a value.

### 3. Gate the asking on the lookup, mechanically

Deciding to check first does not survive a busy afternoon. Make it structural: the tool that files a
question **refuses to file it** until a search of the decision record has run and come back empty.

This is worth building even when it is crude. A keyword search that surfaces three loosely related
past decisions is enough — you are not trying to answer the question automatically, you are trying to
make "has this already been settled?" impossible to skip.

Two properties matter more than the search quality:

- **It must be cheap to run**, or it gets routed around.
- **A hit must be readable in place.** If confirming a match costs a context switch, it will be
  dismissed rather than read.

And keep the inverse rule explicit, because an over-strong gate creates its own failure: **re-raising
a genuinely open question is not a double-ask.** The rule is against asking what was answered, not
against asking what was deferred. If the guard cannot tell those apart, it will train people to
suppress real questions, which costs more than the re-asking did.

### 4. An answer is a hand-back, not a finish

The most common way a decision record rots is that answering closes it.

Someone answers, the item goes to "done", and the work the answer unblocks never happens — because
"done" was recorded by the person who supplied the *input*, not by the person who owed the *output*.
The item is now terminal and invisible, and nothing failed loudly.

So separate the two states:

- **They answered** → the item is *handed back*, and belongs in your inbox.
- **You acted on it** → the item is *closed*, and the closure carries a reference to what you actually
  did: a commit, a file, a link.

A record whose closure names no artifact is an assertion, not evidence. Make the tool refuse a
terminal state that has no reference attached; it costs one line and it removes an entire class of
quiet loss.

## Scan the hand-backs on a schedule

The inbox above only works if something reads it. Once per working session, list every item that is
in a terminal state **and** has no action stamp from you. That query is exact — no heuristics, no time
window — and it catches a hand-back from ten minutes ago and one from ten months ago identically.

Every item leaves that scan in exactly one of four ways:

1. acted on now;
2. scheduled as this session's work;
3. **bounced** — reopened with a specific follow-up question appended;
4. stamped "no action needed", with the reason.

Anything older than about a week in that list is a signal that the underlying loss is reasserting.

⚠ **Terminal does not mean handed back on every channel, and generalising that is its own bug.** A
channel where *their* action is the completion — something they send, approve or reject — is finished
when they act. Pointing the hand-back lens at those will report hundreds of properly closed items as
outstanding, and the resulting noise will get the whole scan ignored. Before applying it, ask what a
terminal state *means* on that particular channel.

## What this costs when it is missing

The visible cost is the repeated question, which is mildly irritating and easy to dismiss as
politeness overhead.

The real cost is that **no single version of the decision is authoritative**, so every future
conversation about it starts by reconstructing it, and each reconstruction is a fresh chance to drift.
Eventually two parts of the system are built to two different readings of the same answer, both
defensible, and nobody can say which was intended — including the person who decided.
