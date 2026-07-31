---
name: model-provenance
description: "A model's self-reported identity is not evidence. Verify which model actually produced an output from the transcript or API metadata, never from what the text claims. Use whenever a second opinion, review or sign-off is attributed to a specific model; whenever a fallback or router may have substituted one; and before recording that 'model X reviewed this'."
---

# A model cannot tell you which model it is

**Asking a model what it is produces a plausible answer, not a verified one.** A model told it is
"Reviewer B" will introduce itself as Reviewer B. A model behind a router that quietly fell back to a
different model will still describe itself using whatever its prompt implied. The self-report is
generated text, subject to exactly the same failure modes as every other generated claim.

This matters the moment an output's *value depends on where it came from*: an independent review, a
second opinion, an adversarial check, a sign-off recorded for later. If that attribution is wrong, the
artifact is not merely mislabelled: the property you were buying, independence, never existed, and
everything downstream inherits a confidence it has not earned.

## The rule

**Verify provenance out-of-band, from a channel the model does not author.**

| trustworthy | not trustworthy |
|---|---|
| the `model` field in the API response | the model saying which model it is |
| the session transcript or request log | a model identifier inside the generated text |
| usage and billing records broken down by model | the name you put in your own prompt |
| | **the value your client resolved before sending** |

The distinction is simple: **did the model write it, or did the infrastructure record it?** Only the
second is evidence.

Two traps in that table are worth naming, because both look like evidence:

- **What your client resolved is a record of what you ASKED FOR, never of what answered.** Silent
  substitution is exactly the case where the two differ, so treating the outgoing value as proof
  licenses the first failure below. It belongs on the right-hand side.
- **A `model` field is only as trustworthy as whatever recorded it.** For a first-party API that is
  the provider. Through an aggregator, a proxy or a gateway, you are trusting that intermediary too:
  which may be fine, but it is a different claim and worth knowing you are making it.

## Three specific ways attribution goes wrong

**1. Silent substitution.** A request for one model is served by another: a router, a quota rule, a
deprecated alias, a capacity event. The response is perfectly good. The identity is not the one you
asked for, and nothing in the text will mention it.

**2. Role-play absorption.** Give a model a persona and it adopts it, including the persona's claimed
identity. Ask it to "review this as an independent model" and you get text that reads as an
independent review, produced by the same weights that wrote the thing under review. This is the
failure that makes the whole exercise pointless while looking entirely successful.

**3. Refusal-shaped emptiness.** A request is declined at a safety layer. The channel returns an error
or an empty completion, and a wrapper that never checks the status records a run that produced
nothing, or worse, retries against a different model and records *that* as the original. A review
with no content is not a review that passed.

## How to actually check

1. **Read the identity from the response metadata or the transcript, not the body.** Put it where the
   output is consumed, so it is one line of code rather than a habit somebody has to remember.
2. **Assert every turn matches.** In a multi-turn exchange, check them all. A run that starts on one
   model and finishes on another is a mixed artifact, and only per-turn inspection reveals it.
3. **Fail closed.** An unreadable transcript, a missing field, an empty completion, a non-success
   status: all mean *unverified*, which must be treated as *not that model*, never as "probably fine".
4. **Record the verified identity beside the output**, with the date. An attribution nobody checked is
   indistinguishable from one that was, a week later.
5. **Attribute honestly when a run is partial.** If the exchange died partway, keep the genuine part
   and say where it ended, rather than describing the whole artifact as that model's work.

## Do not defeat the point

Two temptations, both of which produce a green light and no signal:

- **Rewording a request until a safety layer stops declining it.** That is evasion, and it changes
  what you asked. A review of a disguised artifact tells you about the disguise. If a channel declines
  a subject, that is an answer: find a different reviewer, or do the work yourself and say so.
- **Retrying quietly after a refusal or a substitution.** If you retry, record what actually happened.
  A log saying "reviewed by X" when X declined is worse than no log at all, because it stops anyone
  from looking again.

## The one-sentence version

If your records say a particular model reviewed something, you should be able to point at where that
came from, and it must not be the review itself.
