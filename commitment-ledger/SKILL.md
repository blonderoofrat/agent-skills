---
name: commitment-ledger
description: "Stop work evaporating between sessions. Every promise gets one durable record that resurfaces until it is explicitly disposed of; 'parked' and 'killed' are honest endings and a silent drop is the only failure. Use whenever you write 'we should' or 'later' or 'I'll come back to this', whenever a session ends with loose threads, and whenever you find an old note nobody acted on or cancelled."
---

# A ledger, so nothing is dropped silently

Find a sentence in an old planning document: *we should check whether the contact form is still
forwarding correctly.*

Nobody checked. Nobody decided not to. There was no argument about it, no deprioritisation, no note
saying it turned out to be unnecessary. It was written down in a place where writing things down feels
like doing something about them, and then the document was closed.

Go looking for its siblings and there are always more than you expect. Not abandoned work — work that
was never *decided against*. That is the distinction this skill exists to enforce, and it is the whole
of it: **a dropped commitment and a cancelled one look identical from the outside, and only one of
them is a failure.**

## Why documents cannot hold commitments

A planning document is organised by topic, which is exactly wrong for tracking obligations.

- It is read when you are already working on its topic — that is, when you least need reminding.
- Its commitments are embedded in prose, so no query can enumerate them.
- It has no state. A sentence saying "we should do X" reads identically before and after X is done,
  so a stale document and a current one are indistinguishable.
- Nothing ever *fails* because it went unread. There is no signal.

Any list with those four properties will lose things, however diligently it is written.

## The shape that works

**One record per commitment, in one store, with a status, that resurfaces on a schedule until it is
disposed of.** That is the entire mechanism. The details that make it work:

### Capture at the moment of promising, not later

The instant you write "we should", "later", "once X lands", or "I'll come back to this" — file it.
Filing later means filing what you remember, which is a filtered subset already biased toward what you
were going to do anyway.

Make filing take seconds. One command, a title, a note. If capture costs more than the thought did, it
does not happen on the days it matters.

### Three endings, all legitimate

- **done** — with a reference to the artifact. A closure naming no artifact is an assertion.
- **parked** — not now, with the condition that would revive it. "Parked until we have more than one
  user" is a real answer and a useful one.
- **killed** — decided against, **with the reason**. This is the ending people skip, and it is the most
  valuable of the three.

The reason on a kill is what stops the idea being re-proposed in six months by someone with less
information than you have now. Reading old commitments back and writing *no, and here is why* is not a
failure of follow-through — it is a decision that never got made finally getting made, with the
benefit of everything learned since.

**A silent drop is the only bug.** Not slowness. Not a long list. Not a high kill rate.

### Resurface automatically, at a boundary you cannot skip

Print the open items at the start of every session, or on whatever event genuinely bounces off your
attention. A list you have to remember to open is the planning document again, with extra steps.

Keep the surfacing terse — titles, ages, and counts. A wall of full text at every session start gets
skimmed within a week, and a skimmed reminder is a missing one.

### Measure the age, not the count

The health metric is **the number of open items older than some threshold**, and the only requirement
is that it trends **down**.

Total count is a bad metric: it punishes capture, which is the behaviour you most want. An open item
filed this morning is the system working. An open item from four months ago that has surfaced sixty
times and been passed over sixty times is a decision being avoided, and it should be forced to an
ending.

## Two failure modes to design against

**The ledger becomes a wish list.** If anything anyone mentions gets filed, the open list grows past
the point of being read, and it stops being consulted at all. Guard: file *commitments* — things you
said you would do — not ideas. Ideas belong somewhere with no obligation attached.

**Filing becomes a substitute for doing.** Recording a small task can cost more than the task. Guard:
if it takes under a couple of minutes, do it now; the ledger is for what survives the session.

## Where the real value lands

Not in the completions. The completions would mostly have happened anyway — they are the things you
remembered.

The value is in the **honest nos**: the items that surface, get looked at with fresh information, and
get killed with a written reason. Those are pure gain, because the alternative was not doing them
either — it was not doing them while still carrying them, unresolved, as a low background sense that
there is something outstanding and you cannot name it.
