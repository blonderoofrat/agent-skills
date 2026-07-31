---
name: agent-clean-room
description: "Run a second model as a subagent without your repository's ambient context reaching it. A coding agent injects git branch, changed files and recent commit messages into every subprocess prompt, and that background text is scored alongside your actual question -- so a benign request can be declined because of what your project is about. Use whenever a subagent call is refused, returns nothing, or behaves differently than the same prompt does elsewhere."
---

# Getting a second model to answer, when your repository keeps answering for you

**The symptom:** you ask a second model to review a design, and it declines. The same question, typed
into a fresh chat window, gets a thoughtful answer. Nothing about your question was the problem.

**The cause:** your coding agent does not send your prompt. It sends your prompt *plus* a block of
ambient context: current branch, changed files, recent commit subjects, and usually the contents of
whatever instruction files sit in the project root. A safety classifier scores the **whole** input. If
your repository's subject matter is one it treats cautiously, every subagent call inherits that, no
matter how mundane the actual request.

This is worth naming because the conclusion people reach is *"that model is unusable for me"*, and
they stop trying. The model is fine. The envelope is the problem.

## The fix: launch from somewhere that has nothing to say

Run the second model as a **headless process from a scratch directory outside your repository**: no
version control, no instruction files, no project config. Then hand it a self-contained brief.

```bash
mkdir -p /tmp/review-run          # anywhere OUTSIDE the repo and outside version control
cd /tmp/review-run
# write brief.md here: everything the reviewer needs, and nothing about where it came from
cat brief.md | <your-agent-cli> --print --model <model> "Review this and answer the questions at the end."
```

Three details do the work:

1. **Outside version control.** This is the big one. No repository means no branch name, no diff, no
   commit log in the prompt.
2. **No instruction files.** Project-level agent instructions are auto-loaded from the working
   directory. A scratch directory has none.
3. **Self-contained brief.** The reviewer has no access to your code, so the brief must carry the
   problem whole. This is a feature: it forces you to state the problem clearly, and half the value
   arrives before you send it.

## The line this must not cross

Stripping **irrelevant ambient context** is legitimate: your branch name was never part of your
question. Rewriting the **substance** until a check stops objecting is not.

The reason is practical rather than moral, and it is the reason that should stick: whatever comes
back is a response to what you actually sent. Soften the question and you have a confident answer to
the softened version, which you will then act on as though it addressed the real one. You have not
obtained a review; you have obtained a review of something else.

If a subject is genuinely declined, the two honest moves are to route it to a reviewer that will take
it, or to do the work yourself and record that no second opinion was obtained.

## Learn where your own boundary is, instead of guessing at it

A clean room stops *irrelevant* context causing refusals. It does not tell you which of your **actual
subjects** a given reviewer will engage with, and that is the thing you need in order to plan work.
Guessing wastes runs in both directions: you skip questions that would have been answered, and you
build briefs for questions that were never going to be.

So map it deliberately, once, and write it down:

1. **Probe with small, honest, self-contained briefs** across the subject areas you actually work in.
   One short question each. Frame every one in its own real terms: a probe you had to disguise tells
   you nothing about the real thing.
2. **Record every result, including the refusals**, with the date and the exact framing used. A
   refusal is data, not a failed attempt.
3. **Write down the sample size next to the conclusion.** This is the part people skip and it is the
   part that bites.
4. **Re-probe when a result surprises you.** These boundaries move as models and policies change, so
   the map is a dated snapshot, never a permanent fact.

**Expect the boundary to be about framing as much as topic**, and expect that to mislead you. Two
requests about the same underlying work can land differently depending on whether they are phrased as
a question about a system or a question about a subject. That is a genuinely useful thing to know for
routing, and it is exactly the observation that tempts people across the line in the next section.

**A warning from getting this wrong.** It is very easy to generalise a rule from three or four
successes, publish it to yourself as settled, and then waste a day when the fifth case contradicts it.
Keep the count attached to the claim: *"engaged 4 of 5, all on one afternoon"* is honest and stays
useful; *"it handles this fine"* is neither. Treat the map as a prior for what to attempt, not as a
guarantee, and be readier to revise it than to defend it.

**What the map is for:** deciding **what to ask and who to ask**, routing this question to that
reviewer, and doing the other one yourself. It is a scheduling tool.

**What it is not for:** finding a phrasing that gets a declined subject past the check. If your map
says a subject is declined, the honest conclusions are *use a different reviewer* or *do it yourself
and say so*, never *say it differently until it works*. See the section above: this distinction is
practical, not decorative, because an answer to a disguised question is an answer to the disguise.

## Then verify what actually answered you

A clean-room run puts several layers between you and the thing you invoked, and **the output cannot
tell you which model produced it.** Read the identity from the session transcript or the response
metadata, never from the body, and check every turn.

That problem is general rather than specific to this setup, so it is not restated here: the
companion skill **model-provenance** covers it properly, including why a self-report is worthless and
what to do about a partial run.

What IS specific to launching a headless process, and worth spelling out:

- **Empty success.** The process exits 0 and writes nothing, or writes only an error into a file
  nobody opens. A wrapper that does not check both the exit status *and* the output length will
  record a review that never happened. Treat empty output as failure, always.
- **Dying mid-answer.** A run can produce genuinely good output and then stop partway. Keep the real
  part, say where it ended, and finish the rest yourself, attributed honestly. Do not silently retry
  until something completes and then describe the whole artifact as that model's work.

## Writing the brief so it earns its keep

The mechanics above get you an answer. These get you a *useful* one:

- **State what you already tried and measured, with numbers.** Otherwise you will be told to try it.
- **Name the assumption you want attacked.** The largest wins come from a reviewer rejecting a
  premise you stopped seeing, not from answering the question you asked.
- **Give explicit permission to say the work is not worth doing.** Without it you get a plan, when
  what you needed was an opinion.
- **Do not let the brief flatter you.** If the brief describes your own work, every ambiguous detail
  will quietly resolve in your favour without a single dishonest sentence, and the reviewer inherits
  your blind spot. Quote artifacts, meaning command output, counts and logs, rather than your impression of
  them. Write one line stating the case against yourself. And never assert an absence you have not
  checked: "there was no warning sign" is a claim about evidence and needs evidence.

## When not to bother

If your subagent calls already work, this is pure overhead. Reach for it when a call is refused,
returns nothing, or behaves differently from the same prompt in a normal session: that difference is
the signal that something you did not write is being read alongside what you did.
